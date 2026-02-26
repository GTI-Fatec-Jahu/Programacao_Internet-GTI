# Aula 13 — Segurança e Registro de Usuários

> **Disciplina:** Programação para Internet (ILP951)  
> **Professor:** Ronan Adriel Zenatti  
> **Pré-requisitos:** Aula 12 concluída — navegação mestre-detalhe implementada.

---

## 🗺️ O que você vai aprender nesta aula

Atualmente qualquer pessoa que conheça a URL do sistema consegue criar, editar e excluir dados livremente. Hoje você implementa a primeira camada real de segurança: o **cadastro de usuários com hash de senha usando bcrypt**. Você vai entender por que senhas em texto puro são inaceitáveis, como o hash criptográfico torna um vazamento de banco inútil para o atacante, e vai construir o formulário de registro completo com validações de segurança e um indicador visual de força de senha. Na Aula 14, esse cadastro será a base do sistema de login e proteção de rotas.

---

## Parte 1 — Por que senhas em texto puro são proibidas

### O cenário de um vazamento

Violações de banco de dados acontecem em empresas de todos os tamanhos — desde startups até gigantes como Yahoo e LinkedIn. Quando um banco com senhas em texto puro é comprometido, o atacante tem acesso imediato às credenciais de todos os usuários. Como a maioria das pessoas reutiliza senhas, um único vazamento pode comprometer e-mail, banco e redes sociais simultaneamente.

A solução não é "esconder melhor" as senhas no banco — é garantir que, mesmo com acesso total ao banco, o atacante não consiga recuperar as senhas originais. Isso é feito com **hash criptográfico**.

### O que torna o bcrypt ideal para senhas

O **bcrypt** tem três propriedades que o tornam o padrão para armazenamento de senhas. Primeiro, é **irreversível**: matematicamente impossível recuperar a entrada a partir do hash. Segundo, incorpora um **salt automático**: um valor aleatório diferente é gerado para cada hash, garantindo que dois usuários com a mesma senha tenham hashes completamente distintos — inviabilizando ataques de "rainbow table" (tabelas pré-calculadas de correspondência). Terceiro, é **intencionalmente lento**: o fator de custo (`rounds`) torna cada tentativa de força bruta impraticável em escala.

```
"senha123" + salt_A → $2b$12$xKp8M...Lq7nR  (hash do usuário A)
"senha123" + salt_B → $2b$12$mTq9N...Kp3wS  (hash do usuário B — diferente!)
"Senha123" + salt_C → $2b$12$rYm5J...Vn8xQ  (maiúscula muda o hash completamente)
```

[Diagrama educacional comparativo em dois painéis horizontais. Painel superior "❌ Senha em texto puro": banco de dados estilizado com tabela mostrando coluna 'senha' com valores legíveis 'abc123', 'senha123', 'qwerty'. Uma seta vermelha espessa rotulada "banco vazado →" aponta para ícone de criminoso com expressão satisfeita e nuvem de fala "Tenho todas as senhas!". Painel inferior "✅ Hash bcrypt": mesmo banco mas coluna 'senha_hash' com valores longos e ilegíveis começando com '$2b$12$...'. A mesma seta vermelha "banco vazado →" aponta para o ícone de criminoso com expressão confusa, nuvem de fala "Inútil sem anos de processamento" e um relógio marcando infinito. Fundo branco, flat design, paleta vermelha para o cenário ruim, verde para o correto, legendas em português.]

![Texto puro entrega as senhas imediatamente; bcrypt torna o vazamento do banco inútil para o atacante](../imgs/Aula_13_img_01.png)

---

## Parte 2 — Instalando e usando bcrypt

```
pip install bcrypt
pip freeze > requirements.txt
```

A API do bcrypt tem apenas duas funções que você vai usar: `hashpw` para gerar o hash no cadastro e `checkpw` para verificar a senha no login.

```python
import bcrypt

# ── Gerar hash (usado no CADASTRO) ───────────────────────────────────────
senha_original = "minhaSenha@123"

# bcrypt trabalha com bytes — encode() converte string para bytes
# gensalt() gera o salt aleatório automaticamente a cada chamada
hash_bytes = bcrypt.hashpw(senha_original.encode('utf-8'), bcrypt.gensalt())

# O hash é bytes; para guardar no banco (VARCHAR), convertemos para string
hash_string = hash_bytes.decode('utf-8')
# Exemplo de resultado: '$2b$12$KIX9Pmt2mL9bFuHk9HMOeOC8jkM5qXg0...'

# ── Verificar senha (usado no LOGIN) ────────────────────────────────────
senha_tentativa = "minhaSenha@123"
hash_do_banco   = hash_string   # recuperado com SELECT da tabela usuario

resultado = bcrypt.checkpw(
    senha_tentativa.encode('utf-8'),
    hash_do_banco.encode('utf-8')
)
# checkpw extrai o salt do hash e repete o processo — retorna True ou False
print(resultado)  # True

# Tentativa com senha errada
resultado_errado = bcrypt.checkpw(
    "senhaErrada".encode('utf-8'),
    hash_do_banco.encode('utf-8')
)
print(resultado_errado)  # False
```

---

## Parte 3 — Criando a tabela de usuários

No `db_setup.py`, adicione a tabela `usuario`:

```python
cursor.execute('''
    CREATE TABLE IF NOT EXISTS usuario (
        id            INT AUTO_INCREMENT PRIMARY KEY,
        nome          VARCHAR(200)  NOT NULL,
        email         VARCHAR(200)  NOT NULL UNIQUE,
        senha_hash    VARCHAR(255)  NOT NULL,
        perfil        ENUM('usuario','editor','admin') DEFAULT 'usuario',
        ativo         TINYINT(1)    NOT NULL DEFAULT 1,
        criado_em     TIMESTAMP     DEFAULT CURRENT_TIMESTAMP,
        ultimo_login  TIMESTAMP     NULL
    )
''')
conn.commit()
print('✅ Tabela "usuario" criada.')
```

Os detalhes de design importantes: `email UNIQUE` — o banco impede duplicatas automaticamente, dispensando verificação manual para esse campo; `senha_hash VARCHAR(255)` — suficiente para qualquer hash bcrypt atual; `ultimo_login TIMESTAMP NULL` — começa como NULL e será preenchido na Aula 14 a cada autenticação bem-sucedida; `perfil ENUM` — apenas valores definidos são aceitos, prevenindo dados inválidos.

---

## Parte 4 — Rota de registro completa

### Exemplo prático 1 — Validações de segurança no servidor

```python
import bcrypt
from flask import Flask, render_template, request, flash, redirect, url_for
from db import execute_query

@app.route('/registro', methods=['GET', 'POST'])
def registro():
    if request.method == 'POST':

        nome     = request.form.get('nome', '').strip()
        email    = request.form.get('email', '').strip().lower()
        # .lower() normaliza o e-mail — "Joao@FATEC.br" e "joao@fatec.br"
        # são tratados como o mesmo endereço
        senha    = request.form.get('senha', '')
        confirma = request.form.get('confirma_senha', '')

        # ── Validações ───────────────────────────────────────────────
        erros = []

        if not nome or len(nome) < 3:
            erros.append('O nome deve ter ao menos 3 caracteres.')

        if not email or '@' not in email or '.' not in email.split('@')[-1]:
            erros.append('Informe um e-mail válido.')

        # Regras de senha — validação no servidor, não apenas no cliente
        if len(senha) < 8:
            erros.append('A senha deve ter ao menos 8 caracteres.')
        if not any(c.isupper() for c in senha):
            erros.append('A senha deve conter ao menos uma letra maiúscula.')
        if not any(c.isdigit() for c in senha):
            erros.append('A senha deve conter ao menos um número.')
        if senha != confirma:
            erros.append('As senhas não coincidem.')

        # Verifica duplicidade de e-mail antes de tentar inserir
        # Mais claro que depender do erro UNIQUE do banco
        if not erros and email:
            existe = execute_query(
                'SELECT id FROM usuario WHERE email = %s', (email,), fetch=True
            )
            if existe:
                erros.append('Este e-mail já está cadastrado. '
                             'Use outro e-mail ou faça login.')

        if erros:
            for e in erros:
                flash(e, 'danger')
            return render_template('registro.html', nome=nome, email=email)

        # ── Hash e inserção ──────────────────────────────────────────
        senha_hash = bcrypt.hashpw(
            senha.encode('utf-8'),
            bcrypt.gensalt()  # gensalt() usa custo 12 por padrão
        ).decode('utf-8')

        execute_query(
            'INSERT INTO usuario (nome, email, senha_hash) VALUES (%s, %s, %s)',
            (nome, email, senha_hash)
        )

        flash('Conta criada com sucesso! Faça login para continuar.', 'success')
        return redirect(url_for('login'))

    return render_template('registro.html')
```

### Exemplo prático 2 — Template com indicador de força de senha

```html
{% extends 'base.html' %}
{% block titulo %}Criar Conta{% endblock %}

{% block conteudo %}
<div class="row justify-content-center">
  <div class="col-md-6 col-lg-5">
    <div class="card shadow-sm">

      <div class="card-header bg-success text-white text-center py-3">
        <h4 class="mb-0">🔐 Criar Nova Conta</h4>
      </div>

      <div class="card-body p-4">
        <form action="{{ url_for('registro') }}" method="post">

          <div class="mb-3">
            <label for="nome" class="form-label">Nome completo</label>
            <input type="text" class="form-control" id="nome" name="nome"
                   value="{{ nome | default('') }}"
                   required minlength="3"
                   placeholder="Seu nome completo">
          </div>

          <div class="mb-3">
            <label for="email" class="form-label">E-mail</label>
            <input type="email" class="form-control" id="email" name="email"
                   value="{{ email | default('') }}"
                   required placeholder="seu@email.com">
          </div>

          <div class="mb-3">
            <label for="senha" class="form-label">Senha</label>
            <input type="password" class="form-control" id="senha" name="senha"
                   required minlength="8"
                   placeholder="Mínimo 8 caracteres"
                   oninput="avaliarForca(this.value)">
            {# type="password": caracteres ocultos — nunca use type="text" para senhas #}

            {# Barra visual de força de senha — atualizada via JavaScript #}
            <div class="progress mt-2" style="height:5px">
              <div class="progress-bar" id="barraForca"
                   role="progressbar" style="width:0%"></div>
            </div>
            <small id="labelForca" class="text-muted d-block mt-1"></small>
          </div>

          <div class="mb-3">
            <label for="confirma_senha" class="form-label">Confirmar senha</label>
            <input type="password" class="form-control" id="confirma_senha"
                   name="confirma_senha" required
                   placeholder="Repita a senha acima">
          </div>

          <button type="submit" class="btn btn-success w-100 py-2">
            ✅ Criar Conta
          </button>

        </form>

        <hr class="my-3">
        <p class="text-center mb-0 small">
          Já tem conta?
          <a href="{{ url_for('login') }}">Fazer login →</a>
        </p>
      </div>
    </div>
  </div>
</div>
{% endblock %}

{% block scripts %}
<script>
// Avaliador de força de senha — apenas orientação visual para o usuário
// A validação real continua sendo feita no servidor Python
function avaliarForca(senha) {
    const barra  = document.getElementById('barraForca');
    const label  = document.getElementById('labelForca');

    let pontos = 0;
    if (senha.length >= 8)             pontos++;   // comprimento mínimo
    if (senha.length >= 12)            pontos++;   // comprimento bom
    if (/[A-Z]/.test(senha))           pontos++;   // tem maiúscula
    if (/[0-9]/.test(senha))           pontos++;   // tem número
    if (/[^A-Za-z0-9]/.test(senha))   pontos++;   // tem símbolo especial

    const niveis = [
        { pct:   0, cor: '',              texto: '' },
        { pct:  20, cor: 'bg-danger',     texto: 'Muito fraca' },
        { pct:  40, cor: 'bg-warning',    texto: 'Fraca' },
        { pct:  60, cor: 'bg-info',       texto: 'Moderada' },
        { pct:  80, cor: 'bg-primary',    texto: 'Forte' },
        { pct: 100, cor: 'bg-success',    texto: 'Muito forte ✓' },
    ];

    const n = niveis[pontos];
    barra.style.width = n.pct + '%';
    barra.className   = 'progress-bar ' + n.cor;
    label.textContent = n.texto;
}
</script>
{% endblock %}
```

### Exemplo prático 3 — Gerenciamento de usuários (área admin)

```python
@app.route('/admin/usuarios')
def admin_usuarios():
    # Rota protegida por login — proteção adicionada na Aula 14
    usuarios = execute_query(
        '''SELECT id, nome, email, perfil, ativo, criado_em, ultimo_login
           FROM usuario ORDER BY nome''',
        fetch=True
    )
    return render_template('admin_usuarios.html', usuarios=usuarios)


@app.route('/admin/usuarios/<int:id>/toggle', methods=['POST'])
def toggle_usuario(id):
    """Ativa ou desativa um usuário (soft delete de conta)."""
    resultado = execute_query(
        'SELECT nome, ativo FROM usuario WHERE id = %s', (id,), fetch=True
    )
    if not resultado:
        flash('Usuário não encontrado.', 'warning')
        return redirect(url_for('admin_usuarios'))

    u = resultado[0]
    novo = 0 if u['ativo'] else 1
    execute_query(
        'UPDATE usuario SET ativo = %s WHERE id = %s', (novo, id)
    )
    verbo = 'ativado' if novo else 'desativado'
    flash(f'Usuário "{u["nome"]}" {verbo}.', 'success')
    return redirect(url_for('admin_usuarios'))
```

```html
{# templates/admin_usuarios.html #}
{% extends 'base.html' %}
{% block titulo %}Usuários — Admin{% endblock %}

{% block conteudo %}
<div class="d-flex justify-content-between align-items-center mb-3">
  <h2>👥 Usuários do Sistema</h2>
  <a href="{{ url_for('registro') }}" class="btn btn-success btn-sm">
    ➕ Novo Usuário
  </a>
</div>

<div class="table-responsive">
  <table class="table table-hover table-bordered align-middle">
    <thead class="table-dark">
      <tr>
        <th>#</th><th>Nome</th><th>E-mail</th><th>Perfil</th>
        <th>Último login</th><th>Status</th><th>Ação</th>
      </tr>
    </thead>
    <tbody>
      {% for u in usuarios %}
      <tr class="{% if not u.ativo %}table-secondary text-muted{% endif %}">
        <td>{{ u.id }}</td>
        <td>{{ u.nome }}</td>
        <td>{{ u.email }}</td>
        <td>
          {% if u.perfil == 'admin' %}
            <span class="badge bg-danger">Admin</span>
          {% elif u.perfil == 'editor' %}
            <span class="badge bg-warning text-dark">Editor</span>
          {% else %}
            <span class="badge bg-secondary">Usuário</span>
          {% endif %}
        </td>
        <td class="small text-muted">
          {{ u.ultimo_login if u.ultimo_login else 'Nunca' }}
        </td>
        <td>
          {% if u.ativo %}
            <span class="badge bg-success">Ativo</span>
          {% else %}
            <span class="badge bg-secondary">Inativo</span>
          {% endif %}
        </td>
        <td>
          <form action="{{ url_for('toggle_usuario', id=u.id) }}"
                method="post" style="display:inline">
            <button type="submit"
                    class="btn btn-sm {% if u.ativo %}btn-outline-danger{% else %}btn-outline-success{% endif %}">
              {% if u.ativo %}Desativar{% else %}Ativar{% endif %}
            </button>
          </form>
        </td>
      </tr>
      {% endfor %}
    </tbody>
  </table>
</div>
{% endblock %}
```

---

## Atividade da Aula

Crie a tabela `usuario` no `db_setup.py` e implemente a rota `/registro` com validação completa no servidor: tamanho mínimo, presença de maiúscula e número, confirmação de senha e verificação de e-mail único no banco. Gere o hash bcrypt antes de gravar — nunca a senha original. Adicione o indicador visual de força de senha com JavaScript. Crie a rota `/admin/usuarios` com a listagem e o botão de ativar/desativar. Teste três cenários: cadastro válido, e-mail duplicado, e senhas que não coincidem.

```
git add .
git commit -m "Aula 13: registro de usuários com hash bcrypt e validação"
git push
```

---

## Resumo da Aula

Você construiu a primeira camada real de segurança do sistema. Entendeu por que bcrypt é a escolha correta para senhas — irreversível, com salt automático e custo computacional ajustável. Implementou validações robustas no servidor (tamanho, maiúsculas, números, confirmação, e-mail único) e gravou apenas o hash no banco, nunca a senha original. Na próxima aula, esse cadastro ganha vida completa: você implementa o login, a sessão e a proteção de rotas com `@login_required`.

---

> ⬅️ [Aula anterior: Visualização Mestre-Detalhe](Aula_12_Visualizacao_Mestre_Detalhe.md) | ➡️ [Próxima Aula: Login e Controle de Sessão](Aula_14_Login_e_Controle_de_Sessao.md)
