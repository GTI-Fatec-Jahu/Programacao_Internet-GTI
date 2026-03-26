# 📋 Avaliação Prática — A1
## Disciplina: Programação para Internet (ILP951)
**Professor:** Ronan Adriel Zenatti · **FATEC Jahu** · 1º Semestre / 2026

---

> ⚠️ **Esta avaliação não será entregue. Serve exclusivamente para autoavaliação e fixação dos conhecimentos adquiridos nas Aulas 01 a 08.**  
> Não há dicas, exemplos de código ou respostas neste documento. Utilize apenas o que foi aprendido em aula.

---

## 🎯 Descrição do Projeto

Desenvolva uma aplicação web com **Python e Flask** que funcione como uma **Calculadora de Gorjeta**. A aplicação deve:

- Receber do usuário o **valor total da conta**, a **quantidade de pessoas** e o **percentual de gorjeta** desejado
- Calcular o **valor da gorjeta**, o **valor total com gorjeta** e o **valor que cada pessoa deve pagar**
- Classificar o pagador conforme o percentual informado:
  - **Mão de vaca** — gorjeta abaixo de 5%
  - **Legal** — gorjeta de 5% até 15% (inclusive)
  - **Generoso** — gorjeta acima de 15%
- Apresentar os resultados de forma clara, estilizada com **Bootstrap**
- Ter o código versionado e publicado no **GitHub**

---

## 🗺️ Roteiro de Desenvolvimento

Siga esta sequência lógica do início ao fim. Cada tópico representa uma etapa que deve estar concluída antes de avançar para o próximo.

---

### Etapa 1 — Preparação do Ambiente

1. Crie uma pasta local para o projeto com um nome adequado
2. Abra a pasta no Visual Studio Code
3. Crie e ative o ambiente virtual Python dentro da pasta do projeto
4. Instale o Flask com o gerenciador de pacotes do Python
5. Gere o arquivo de dependências do projeto
6. Crie o arquivo `.gitignore` com as entradas necessárias para ignorar o ambiente virtual e os arquivos de cache do Python
7. Inicialize o repositório Git local
8. Faça o primeiro commit com a estrutura inicial

---

### Etapa 2 — Estrutura de Pastas do Projeto

1. Crie a estrutura de diretórios padrão de um projeto Flask:
   - Pasta para os templates HTML
   - Pasta para os arquivos estáticos
   - Subpastas para CSS e JavaScript dentro da pasta de estáticos
2. Crie o arquivo principal da aplicação Flask na raiz do projeto
3. Confirme que a estrutura está organizada antes de prosseguir

---

### Etapa 3 — Template Base com Bootstrap

1. Crie o template base que será herdado por todas as páginas da aplicação
2. O template base deve conter:
   - Estrutura HTML5 válida e completa
   - Integração com o Bootstrap via CDN
   - Uma barra de navegação (navbar) com o nome da aplicação
   - Um bloco de conteúdo que as páginas filhas irão preencher
   - Um rodapé com informações do desenvolvedor
   - Um bloco para scripts JavaScript adicionais
3. Inclua o mecanismo para exibir flash messages de feedback ao usuário

---

### Etapa 4 — Página Inicial (Formulário)

1. Crie o template da página inicial herdando do template base
2. A página deve apresentar um formulário com os seguintes campos:
   - **Valor total da conta** — campo numérico, aceita valores decimais, obrigatório
   - **Quantidade de pessoas** — campo numérico, apenas inteiros positivos, obrigatório
   - **Percentual de gorjeta (%)** — campo numérico, aceita valores decimais, obrigatório
3. Todos os campos devem ter `label` descritivo e `placeholder` com exemplo de valor
4. O formulário deve ter um botão de envio e um botão para limpar os campos
5. Aplique as classes do Bootstrap para estilizar o formulário de forma profissional
6. Organize o layout com o sistema de grid do Bootstrap onde fizer sentido

---

### Etapa 5 — Lógica de Cálculo no Back-end

1. No arquivo principal do Flask, crie a rota para a página inicial que aceite os métodos GET e POST
2. No bloco de processamento do POST:
   - Colete os três valores enviados pelo formulário
   - Aplique validação no servidor para cada campo:
     - Verifique se os campos estão preenchidos
     - Verifique se os valores numéricos são válidos (trate possíveis erros de conversão)
     - Verifique se o valor da conta é maior que zero
     - Verifique se a quantidade de pessoas é um número inteiro maior que zero
     - Verifique se o percentual de gorjeta é um número maior ou igual a zero
   - Se houver erros, exiba as mensagens de erro via flash e re-renderize o formulário com os dados já digitados
3. Se os dados forem válidos, realize os cálculos:
   - Calcule o valor da gorjeta
   - Calcule o valor total (conta + gorjeta)
   - Calcule o valor por pessoa (total dividido pela quantidade)
   - Determine a classificação conforme o percentual informado
4. Decida a estratégia de exibição dos resultados (redirecionar para outra rota ou renderizar diretamente com os dados)

---

### Etapa 6 — Página de Resultados

1. Crie o template da página de resultados herdando do template base
2. A página deve exibir de forma clara e organizada:
   - O **valor da gorjeta** calculado
   - O **valor total** da conta com gorjeta
   - O **valor por pessoa**
   - A **classificação** do pagador, com destaque visual diferente para cada categoria:
     - Mão de vaca (abaixo de 5%) — use cor ou estilo que transmita a ideia
     - Legal (5% a 15%) — use cor ou estilo neutro/positivo
     - Generoso (acima de 15%) — use cor ou estilo que valorize a generosidade
3. Use componentes do Bootstrap (cards, badges, alertas ou tabelas) para organizar os dados
4. Inclua um link ou botão que permita ao usuário realizar um novo cálculo
5. Certifique-se de que os valores monetários são exibidos com duas casas decimais

---

### Etapa 7 — Estilo Personalizado

1. Crie o arquivo CSS próprio da aplicação na pasta de estáticos
2. Adicione ao menos três regras CSS personalizadas que complementem o Bootstrap:
   - Um estilo para o rodapé
   - Um estilo para a navbar ou para o cabeçalho da página de resultados
   - Um estilo para o destaque de cada classificação de gorjeta
3. Referencie o arquivo CSS no template base usando a função correta do Flask para arquivos estáticos
4. Certifique-se de que o arquivo CSS é carregado após o Bootstrap

---

### Etapa 8 — Testes e Verificação

Antes de publicar, teste manualmente os seguintes cenários:

1. **Cenário válido padrão** — preencha os três campos com valores corretos e verifique se os cálculos estão corretos
2. **Gorjeta mão de vaca** — informe um percentual abaixo de 5% e verifique se a classificação correta é exibida
3. **Gorjeta legal** — informe um percentual entre 5% e 15% e verifique a classificação
4. **Gorjeta generoso** — informe um percentual acima de 15% e verifique a classificação
5. **Campos vazios** — tente enviar o formulário sem preencher algum campo e verifique se as mensagens de erro aparecem
6. **Valor inválido** — informe texto em vez de número em algum campo e verifique se o erro é tratado adequadamente
7. **Quantidade de pessoas igual a zero ou negativa** — verifique se a validação impede o cálculo
8. **Responsividade** — redimensione a janela do navegador e verifique se o layout se adapta

---

### Etapa 9 — Publicação no GitHub

1. Verifique se o arquivo `.gitignore` está correto e se a pasta do ambiente virtual não está sendo rastreada pelo Git
2. Verifique se o arquivo de dependências está atualizado
3. Faça commits incrementais de cada etapa concluída — não suba tudo em um único commit
4. Crie um repositório público no GitHub com um nome descritivo
5. Conecte o repositório local ao repositório remoto
6. Envie todos os commits para o GitHub
7. Acesse o repositório no navegador e confirme que todos os arquivos foram enviados corretamente
8. Verifique se é possível clonar o repositório em outra pasta e executar a aplicação a partir do `requirements.txt`

---

## ✅ Checklist de Autoavaliação

Use esta lista para verificar se o projeto está completo antes de considerar a avaliação concluída.

### Estrutura e organização
- [ ] Pasta do projeto com estrutura padrão Flask (`templates/`, `static/`, subpastas)
- [ ] Arquivo principal da aplicação na raiz do projeto
- [ ] `requirements.txt` atualizado com as dependências
- [ ] `.gitignore` com `venv/` e `__pycache__/`
- [ ] Repositório Git com commits distribuídos ao longo do desenvolvimento

### Templates e interface
- [ ] Template base (`base.html`) com navbar, bloco de conteúdo e rodapé
- [ ] Integração correta com Bootstrap via CDN
- [ ] Flash messages visíveis no template base
- [ ] Página inicial com formulário e os três campos obrigatórios
- [ ] Página de resultados herdando do template base
- [ ] Todos os links e referências a arquivos estáticos usando `url_for`
- [ ] Arquivo CSS próprio com ao menos três regras personalizadas

### Funcionalidade
- [ ] Formulário enviando dados via POST
- [ ] Validação no servidor para todos os campos
- [ ] Re-população do formulário com os dados já digitados em caso de erro
- [ ] Cálculos corretos: gorjeta, total e valor por pessoa
- [ ] Classificação correta para os três intervalos de porcentagem
- [ ] Destaque visual diferente para cada classificação
- [ ] Valores exibidos com duas casas decimais

### Publicação
- [ ] Repositório público no GitHub
- [ ] Todos os arquivos presentes no repositório remoto
- [ ] Ambiente virtual **não** enviado para o GitHub

---

## 📐 Referência dos Cálculos

Para verificar se sua lógica está correta, utilize a tabela abaixo:

| Conta (R$) | Pessoas | Gorjeta (%) | Valor Gorjeta | Total | Por Pessoa | Classificação |
|------------|---------|-------------|---------------|-------|------------|---------------|
| 100,00 | 2 | 10 | 10,00 | 110,00 | 55,00 | Legal |
| 200,00 | 4 | 3 | 6,00 | 206,00 | 51,50 | Mão de vaca |
| 150,00 | 3 | 20 | 30,00 | 180,00 | 60,00 | Generoso |
| 80,00 | 1 | 5 | 4,00 | 84,00 | 84,00 | Legal |
| 300,00 | 5 | 15 | 45,00 | 345,00 | 69,00 | Legal |

> 💡 **Atenção aos limites:** gorjeta de exatamente **5%** é classificada como *Legal*, e gorjeta de exatamente **15%** também é classificada como *Legal*. O limite inferior de *Mão de vaca* é **estritamente abaixo de 5%**, e o limite inferior de *Generoso* é **estritamente acima de 15%**.

---

## 📚 Conteúdo das Aulas que Cobre este Projeto

| Aula | Tema | Relevância para este projeto |
|------|------|------------------------------|
| Aula 01 | Introdução, Git e HTML5 | Criação do repositório, estrutura HTML5, `.gitignore` |
| Aula 02 | Flask e Bootstrap | Instalação do Flask, rotas básicas, integração Bootstrap, arquivos estáticos |
| Aula 03 | Templates Jinja2 e Rotas | Template base com herança, `{% block %}`, `{% extends %}`, `url_for` |
| Aula 04 | Formulários e HTTP | Método POST, `request.form`, validação no servidor, padrão PRG, flash messages |

---

> *"A melhor forma de aprender é fazer. Tente resolver cada etapa por conta própria antes de consultar o material das aulas."*  
> Bom trabalho! 🚀
