# Aula 12 — Visualização Mestre-Detalhe

> **Disciplina:** Programação para Internet (ILP951)  
> **Professor:** Ronan Adriel Zenatti  
> **Pré-requisitos:** Aula 11 concluída — CRUD relacional completo com FK e JOIN.

---

## 🗺️ O que você vai aprender nesta aula

Com as operações CRUD funcionando sobre o modelo relacional, hoje você vai aprender a **exibir essa hierarquia de forma elegante e navegável na interface**. O padrão **Mestre-Detalhe** organiza a experiência em dois níveis: uma lista de registros principais (o mestre) e uma página dedicada a um único registro, mostrando seus dados completos e todos os filhos relacionados (o detalhe). Você vai construir cards de estatísticas, abas Bootstrap para múltiplos tipos de dados, breadcrumb de navegação e acordeões para conteúdo agrupado.

---

## Parte 1 — O padrão Mestre-Detalhe

### Onde esse padrão aparece na vida real

O Mestre-Detalhe é ubíquo em sistemas de gestão: o Gmail lista conversas (mestre) e ao clicar exibe as mensagens da conversa (detalhe). Um ERP lista pedidos de venda (mestre) e ao selecionar um exibe os itens, o cliente e o histórico de status (detalhe). O GitHub lista repositórios (mestre) e ao entrar em um exibe código, issues e pull requests (detalhe).

```mermaid
flowchart LR
    subgraph Mestre "/clientes"
        M["📋 Lista\n──────────────────\nJoão Silva  │ 3 pedidos │ R$ 4.200\nMaria Souza │ 1 pedido  │ R$ 800\nCarlos Lima │ 0 pedidos │ R$ 0"]
    end
    subgraph Detalhe "/clientes/1"
        D1["👤 João Silva\nEmail · Cidade · Data cadastro\n─────────────────────────────\n📊 Stats: 3 pedidos · R$ 4.200\n─────────────────────────────\n🛒 Pedido #1 │ R$ 3.499 │ Pago\n🛒 Pedido #2 │ R$   299 │ Aberto\n🛒 Pedido #3 │ R$   189 │ Cancelado"]
    end
    M -- "Clica em João →" --> D1
    D1 -- "← Voltar" --> M
```

[Ilustração educacional mostrando o padrão mestre-detalhe em dois painéis de navegador lado a lado conectados por uma seta. Painel esquerdo rotulado "/clientes": tabela Bootstrap com três linhas, cada uma com nome do cliente, badge com contagem de pedidos e valor total, e botão azul "Ver". A linha "João Silva" está em hover com fundo azul claro. A seta curva rotulada "clica em 'Ver' →" leva ao painel direito rotulado "/clientes/1": navbar no topo, breadcrumb "Início > Clientes > João Silva", bloco de dados do cliente, três cards de estatísticas side-by-side (3 Pedidos, 1 Pago, R$ 4.200), e tabela de pedidos com três linhas e badges coloridos de status. Seta "← Voltar" na parte inferior. Fundo branco, estilo screenshot Bootstrap educacional, legendas em português.]

![O padrão Mestre-Detalhe: a lista navega para o registro completo com seus dados e filhos relacionados](../imgs/Aula_12_img_01.png)

---

## Parte 2 — Construindo a página de detalhe

### Duas queries: dados do pai + lista de filhos

A rota de detalhe faz duas consultas separadas ao banco. Mantê-las separadas é uma escolha pedagógica — facilita ler, testar e depurar cada parte individualmente. Em sistemas de alta performance, poderia-se unificar, mas para fins educacionais a clareza vale mais.

```python
@app.route('/clientes/<int:id>')
def detalhe_cliente(id):

    # ── Query 1: dados do cliente ────────────────────────────────────
    resultado = execute_query(
        'SELECT * FROM cliente WHERE id = %s', (id,), fetch=True
    )
    if not resultado:
        flash('Cliente não encontrado.', 'warning')
        return redirect(url_for('lista_clientes'))
    cliente = resultado[0]

    # ── Query 2: pedidos deste cliente ───────────────────────────────
    pedidos = execute_query(
        '''SELECT id, valor_total, status, criado_em
           FROM pedido
           WHERE cliente_id = %s
           ORDER BY criado_em DESC''',
        (id,), fetch=True
    )

    # ── Estatísticas calculadas em Python ───────────────────────────
    # Usar Python para cálculos simples evita subconsultas SQL complexas
    total_pedidos = len(pedidos)
    valor_total   = sum(float(p['valor_total']) for p in pedidos)
    pedidos_pagos = sum(1 for p in pedidos if p['status'] == 'pago')
    pedidos_abertos = sum(1 for p in pedidos if p['status'] == 'aberto')

    return render_template('cliente_detalhe.html',
                           cliente=cliente,
                           pedidos=pedidos,
                           total_pedidos=total_pedidos,
                           valor_total=valor_total,
                           pedidos_pagos=pedidos_pagos,
                           pedidos_abertos=pedidos_abertos)
```

### Exemplo prático 1 — Template de detalhe com cards de estatísticas

```html
{% extends 'base.html' %}
{% block titulo %}{{ cliente.nome }} — Detalhe{% endblock %}

{% block conteudo %}

{# ── Breadcrumb de navegação ─────────────────────────────────────── #}
<nav aria-label="breadcrumb" class="mb-3">
  <ol class="breadcrumb">
    <li class="breadcrumb-item">
      <a href="{{ url_for('pagina_inicial') }}">Início</a>
    </li>
    <li class="breadcrumb-item">
      <a href="{{ url_for('lista_clientes') }}">Clientes</a>
    </li>
    <li class="breadcrumb-item active" aria-current="page">
      {{ cliente.nome }}
    </li>
  </ol>
</nav>

{# ── Cabeçalho do registro pai ───────────────────────────────────── #}
<div class="d-flex justify-content-between align-items-start mb-4">
  <div>
    <h2 class="mb-1">{{ cliente.nome }}</h2>
    <p class="text-muted mb-1">
      <span class="me-3">📧 {{ cliente.email }}</span>
      {% if cliente.cidade %}
        <span class="badge bg-secondary">{{ cliente.cidade }}</span>
      {% endif %}
    </p>
    <small class="text-muted">
      Cadastrado em {{ cliente.criado_em }}
    </small>
  </div>
  <div class="d-flex gap-2">
    <a href="{{ url_for('editar_cliente', id=cliente.id) }}"
       class="btn btn-outline-warning btn-sm">✏️ Editar</a>
    <a href="{{ url_for('lista_clientes') }}"
       class="btn btn-outline-secondary btn-sm">← Clientes</a>
  </div>
</div>

{# ── Cards de estatísticas ───────────────────────────────────────── #}
<div class="row mb-4">
  <div class="col-6 col-md-3 mb-3">
    <div class="card text-center border-0 bg-primary bg-opacity-10">
      <div class="card-body py-3">
        <div class="fs-2 fw-bold text-primary">{{ total_pedidos }}</div>
        <div class="text-muted small">Pedidos</div>
      </div>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card text-center border-0 bg-success bg-opacity-10">
      <div class="card-body py-3">
        <div class="fs-2 fw-bold text-success">{{ pedidos_pagos }}</div>
        <div class="text-muted small">Pagos</div>
      </div>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card text-center border-0 bg-warning bg-opacity-10">
      <div class="card-body py-3">
        <div class="fs-2 fw-bold text-warning">{{ pedidos_abertos }}</div>
        <div class="text-muted small">Em aberto</div>
      </div>
    </div>
  </div>
  <div class="col-6 col-md-3 mb-3">
    <div class="card text-center border-0 bg-info bg-opacity-10">
      <div class="card-body py-3">
        <div class="fs-2 fw-bold text-info">
          R$ {{ "%.2f"|format(valor_total) }}
        </div>
        <div class="text-muted small">Total gasto</div>
      </div>
    </div>
  </div>
</div>

{# ── Abas para organizar conteúdo relacionado ────────────────────── #}
<ul class="nav nav-tabs mb-0" id="clienteTabs" role="tablist">
  <li class="nav-item" role="presentation">
    <button class="nav-link active" id="tab-pedidos"
            data-bs-toggle="tab" data-bs-target="#aba-pedidos"
            type="button" role="tab">
      🛒 Pedidos
      <span class="badge bg-secondary ms-1">{{ total_pedidos }}</span>
    </button>
  </li>
  <li class="nav-item" role="presentation">
    <button class="nav-link" id="tab-info"
            data-bs-toggle="tab" data-bs-target="#aba-info"
            type="button" role="tab">
      📋 Informações
    </button>
  </li>
</ul>

<div class="tab-content border border-top-0 rounded-bottom p-3 mb-4">

  {# ── Aba Pedidos ────────────────────────────────────────────────── #}
  <div class="tab-pane fade show active" id="aba-pedidos" role="tabpanel">

    <div class="d-flex justify-content-end mb-2 mt-1">
      <a href="{{ url_for('novo_pedido') }}" class="btn btn-sm btn-success">
        ➕ Novo Pedido
      </a>
    </div>

    {% if pedidos %}
    <div class="table-responsive">
      <table class="table table-sm table-hover align-middle mb-0">
        <thead class="table-light">
          <tr>
            <th>#</th>
            <th>Valor</th>
            <th>Status</th>
            <th>Data</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          {% for p in pedidos %}
          <tr>
            <td class="text-muted">{{ p.id }}</td>
            <td class="fw-bold">R$ {{ "%.2f"|format(p.valor_total) }}</td>
            <td>
              {% if p.status == 'pago' %}
                <span class="badge bg-success">Pago</span>
              {% elif p.status == 'aberto' %}
                <span class="badge bg-primary">Aberto</span>
              {% else %}
                <span class="badge bg-danger">Cancelado</span>
              {% endif %}
            </td>
            <td class="text-muted small">{{ p.criado_em }}</td>
            <td>
              <a href="{{ url_for('editar_pedido', id=p.id) }}"
                 class="btn btn-sm btn-outline-warning">✏️</a>
            </td>
          </tr>
          {% endfor %}
        </tbody>
      </table>
    </div>
    {% else %}
    <div class="alert alert-light border mt-2 mb-0">
      Este cliente ainda não tem pedidos.
      <a href="{{ url_for('novo_pedido') }}">Criar primeiro pedido →</a>
    </div>
    {% endif %}

  </div>

  {# ── Aba Informações ────────────────────────────────────────────── #}
  <div class="tab-pane fade" id="aba-info" role="tabpanel">
    <dl class="row mt-2 mb-0">
      <dt class="col-sm-3">Nome</dt>
      <dd class="col-sm-9">{{ cliente.nome }}</dd>

      <dt class="col-sm-3">E-mail</dt>
      <dd class="col-sm-9">{{ cliente.email }}</dd>

      <dt class="col-sm-3">Cidade</dt>
      <dd class="col-sm-9">{{ cliente.cidade | default('Não informado') }}</dd>

      <dt class="col-sm-3">Cadastro</dt>
      <dd class="col-sm-9 text-muted">{{ cliente.criado_em }}</dd>
    </dl>
  </div>

</div>

{% endblock %}
```

[Captura de tela ilustrativa da página de detalhe do cliente renderizada no navegador Bootstrap. Breadcrumb no topo: "Início > Clientes > João Silva". Abaixo, cabeçalho com nome grande "João Silva", email e badge "Jaú" e botões "✏️ Editar" e "← Clientes". Quatro cards de estatísticas lado a lado: "3 Pedidos" (azul), "1 Pagos" (verde), "2 Em aberto" (amarelo), "R$ 4.200,00 Total gasto" (azul claro). Abaixo, abas "🛒 Pedidos (3)" ativa e "📋 Informações". Tabela de pedidos com três linhas: #1 R$3.499 badge verde Pago, #2 R$299 badge azul Aberto, #3 R$189 badge vermelho Cancelado. Estilo screenshot educacional Bootstrap realista.]

![Página de detalhe completa: breadcrumb, cabeçalho, cards de estatísticas, abas e tabela de filhos](../imgs/Aula_12_img_02.png)

---

## Parte 3 — Acordeão para conteúdo agrupado

Quando a lista de filhos é longa ou pode ser agrupada por categoria, o componente acordeão Bootstrap permite exibir seções expansíveis — economizando espaço e melhorando a organização visual.

### Exemplo prático 2 — Pedidos agrupados por status

No `app.py`, adicione o agrupamento antes de passar para o template:

```python
from collections import defaultdict

# Após buscar os pedidos, agrupe por status
por_status = defaultdict(list)
for p in pedidos:
    por_status[p['status']].append(p)

# Ordem de exibição das seções
ordem = ['aberto', 'pago', 'cancelado']
por_status_ordenado = {s: por_status[s] for s in ordem if por_status[s]}

return render_template('cliente_detalhe.html',
                       cliente=cliente,
                       pedidos=pedidos,
                       por_status=por_status_ordenado,
                       ...)
```

No template, substitua a tabela pela versão em acordeão:

```html
{# Acordeão de pedidos agrupados por status #}
<div class="accordion" id="acordeaoPedidos">
  {% for status_nome, grupo in por_status.items() %}

  {# Configura cor do cabeçalho conforme o status #}
  {% set cor_status = {
    'aberto':    'bg-primary text-white',
    'pago':      'bg-success text-white',
    'cancelado': 'bg-danger text-white'
  } %}

  <div class="accordion-item border">
    <h2 class="accordion-header">
      <button
        class="accordion-button {% if not loop.first %}collapsed{% endif %} {{ cor_status[status_nome] }}"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#secao-{{ status_nome }}">
        {{ status_nome | capitalize }} — {{ grupo | length }} pedido(s)
      </button>
    </h2>
    <div id="secao-{{ status_nome }}"
         class="accordion-collapse collapse {% if loop.first %}show{% endif %}">
      <div class="accordion-body p-0">
        <table class="table table-sm mb-0">
          <tbody>
            {% for p in grupo %}
            <tr>
              <td class="text-muted ps-3">#{{ p.id }}</td>
              <td>R$ {{ "%.2f"|format(p.valor_total) }}</td>
              <td class="text-muted small pe-3">{{ p.criado_em }}</td>
            </tr>
            {% endfor %}
          </tbody>
        </table>
      </div>
    </div>
  </div>
  {% endfor %}
</div>
```

---

## Parte 4 — Adicionando links Mestre → Detalhe na listagem

O padrão só funciona se houver uma forma clara de navegar do mestre para o detalhe. Atualize a listagem de clientes para que o nome seja um link clicável:

```html
{# No template clientes.html, dentro do for, coluna Nome #}
<td>
  {# O nome vira um link para o detalhe #}
  <a href="{{ url_for('detalhe_cliente', id=c.id) }}"
     class="text-decoration-none fw-bold">
    {{ c.nome }}
  </a>
  <br>
  <small class="text-muted">{{ c.email }}</small>
</td>

{# E na coluna Ações, adicione o botão de detalhe #}
<td>
  <a href="{{ url_for('detalhe_cliente', id=c.id) }}"
     class="btn btn-sm btn-outline-primary" title="Ver detalhes">👁️</a>
  <a href="{{ url_for('editar_cliente', id=c.id) }}"
     class="btn btn-sm btn-outline-warning ms-1">✏️</a>
</td>
```

---

## Atividade da Aula

Construa a página de detalhe para a entidade pai do seu sistema. Ela deve incluir: breadcrumb com pelo menos três níveis, cards de estatísticas (mínimo: contagem de filhos e soma de algum campo numérico), abas Bootstrap com ao menos duas seções, e tabela de filhos com badges de status coloridos. Adicione o link de navegação na listagem mestre (clique no nome ou botão 👁️). Se o número de filhos puder ser grande, implemente o acordeão agrupado por algum critério que faça sentido para o seu domínio.

```
git add .
git commit -m "Aula 12: padrão mestre-detalhe com cards, abas e breadcrumb"
git push
```

---

## Resumo da Aula

O padrão Mestre-Detalhe transformou a experiência de navegação do sistema: a listagem geral dá uma visão panorâmica com estatísticas rápidas; a página de detalhe aprofunda em um único registro com todos os dados e filhos relacionados. Você usou breadcrumb para orientação, cards de estatísticas para síntese, abas para múltiplas categorias de informação e acordeão para listas longas agrupadas. Na próxima aula, o foco muda para segurança: hash de senhas e registro de usuários.

---

> ⬅️ [Aula anterior: CRUD Relacional](Aula_11_CRUD_Relacional.md) | ➡️ [Próxima Aula: Segurança e Registro](Aula_13_Seguranca_e_Registro.md)
