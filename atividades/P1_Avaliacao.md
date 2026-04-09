# 📋 Avaliação Prática — A1
## Disciplina: Programação para Internet (ILP951)
**Professor:** Ronan Adriel Zenatti · **FATEC Jahu** · 1º Semestre / 2026

---

> ⚠️ **ATENÇÃO: Este é um trabalho avaliativo oficial.** > **Peso na média final:** 3,0 pontos (de um total de 10,0 no semestre).  
> **Formato de entrega:** Submissão do link do repositório público no GitHub através da atividade do Classroom.  
> **Trabalho Individual.** Cópias ou plágios resultarão em nota zero.

---

## 🎯 Descrição do Projeto

Desenvolva uma aplicação web com **Python e Flask** que funcione como um **Simulador de Custos de Viagem**. A aplicação deve:

- Receber do usuário a **distância total da viagem (em km)**, o **consumo médio do veículo (em km/l)** e o **preço do litro do combustível (em R$)**
- Calcular a **quantidade de litros necessários (LN = D / C)**, o **custo total da viagem (CT = LN x P)** e o **custo por quilômetro rodado (CQ = CT / D)**
- Classificar a eficiência do veículo conforme o consumo informado:
  - **Beberrão** — consumo abaixo de 8 km/l
  - **Padrão** — consumo até 15 km/l
  - **Econômico** — consumo até 18 km/l
  - **Super Econômico** — consumo acima de 18 km/l
- Apresentar os resultados de forma clara, estilizada com **Bootstrap**
- Ter o código versionado e publicado no **GitHub**

---

## 📊 Critérios de Avaliação (Total: 3,0 pontos)

A nota será composta pela soma dos critérios abaixo. O não cumprimento de requisitos básicos de estrutura pode zerar etapas inteiras.

| Critério | Descrição | Pontuação Máxima |
| :--- | :--- | :---: |
| **Lógica e Back-end** | Cálculos matemáticos exatos (litros, custo total, custo/km) e aplicação correta da estrutura condicional para a classificação do veículo. | **1,0 pt** |
| **Validação e UX** | Tratamento adequado de erros (campos vazios, valores inválidos, zeros/negativos), uso correto de *flash messages* e manutenção dos dados digitados em caso de erro no formulário. | **0,5 pt** |
| **Interface e Front-end** | Uso correto do Bootstrap, responsividade, hierarquia visual na página de resultados (cores/badges diferentes para cada classificação) e presença de pelo menos três regras CSS próprias. | **0,5 pt** |
| **Arquitetura Flask** | Estrutura de pastas padronizada, uso eficiente do Jinja2 (template base, herança de blocos), roteamento correto (GET/POST) e uso da função `url_for`. | **0,8 pt** |
| **Versionamento (Git/GitHub)** | Repositório público no GitHub com commits incrementais e lógicos, presença do `requirements.txt` atualizado e uso correto do `.gitignore` (a pasta `venv` **não** deve estar no repositório). | **0,2 pt** |

---

## 🗺️ Roteiro de Desenvolvimento Sugerido

Siga esta sequência lógica para garantir que todos os requisitos sejam atendidos.

### Etapa 1 — Preparação e Estrutura
1. Crie a pasta do projeto e o ambiente virtual Python.
2. Instale o Flask, gere o `requirements.txt` e crie o arquivo `.gitignore`.
3. Inicialize o repositório Git local e faça o primeiro commit.
4. Crie a estrutura de diretórios.
5. Crie o arquivo principal da aplicação.

### Etapa 2 — Templates e Front-end
1. Crie o `base.html` com estrutura HTML5, Bootstrap (CDN), navbar, rodapé e bloco para exibição de *flash messages*.
2. Crie a página inicial com o formulário (distância, consumo e preço). Todos os campos devem ser obrigatórios e numéricos.
3. Crie o arquivo CSS próprio na pasta de estáticos, referencie-o no template base e adicione as regras personalizadas exigidas.

### Etapa 3 — Lógica de Roteamento e Cálculo
1. Configure a rota inicial para aceitar GET e POST.
2. Extraia os dados do formulário e realize as validações (verifique se os valores são maiores que zero).
3. Efetue os cálculos necessários.
4. Defina a classificação (Beberrão, Padrão, Econômico ou Super Econômico) baseada no consumo.

### Etapa 4 — Exibição de Resultados
1. Crie o template da página de resultados (herdando de `base.html`).
2. Exiba os valores formatados monetariamente (com duas casas decimais).
3. Utilize componentes do Bootstrap para dar destaque à classificação de eficiência de forma dinâmica (ex: vermelho para Beberrão, verde para Super Econômico).
4. Adicione um botão "Nova Simulação" para voltar ao formulário.

### Etapa 5 — Testes e Publicação
1. Teste casos de erro (formulário vazio, letras no lugar de números) e os três limiares de classificação.
2. Faça o push do seu código para o GitHub.
3. **Verifique** se a pasta do ambiente virtual não foi enviada junto.
4. Submeta o link na atividade do Classroom.

---

## ✅ Checklist de Entrega

Antes de enviar o link do seu projeto, confira se:
- [ ] O `.gitignore` está ignorando o `venv/` e `__pycache__/`
- [ ] O `requirements.txt` existe e está atualizado
- [ ] A aplicação roda sem erros ao executar o arquivo principal
- [ ] O formulário valida os dados corretamente sem "quebrar" a aplicação
- [ ] Os valores de cálculo batem com a tabela de referência abaixo

---

## 📐 Referência dos Cálculos (Para Testes)

| Distância (km) | Consumo (km/l) | Preço (R$) | Litros Necess. | Custo Total (R$) | Custo por km (R$) | Classificação |
|----------------|----------------|------------|----------------|------------------|-------------------|---------------|
| 100 | 10 | 5,00 | 10,00 | 50,00 | 0,50 | Padrão |
| 200 | 6 | 5,50 | 33,33 | 183,33 | 0,92 | Beberrão |
| 300 | 15 | 6,00 | 20,00 | 120,00 | 0,40 | Padrão |
| 340 | 17 | 5,00 | 20,00 | 100,00 | 0,29 | Econômico |
| 400 | 20 | 5,50 | 20,00 | 110,00 | 0,28 | Super Econômico |

> 💡 **Lembrete dos Limites:**
> - Consumo exato de **8 km/l** = *Padrão*
> - Consumo exato de **15 km/l** = *Padrão*
> - Consumo exato de **18 km/l** = *Econômico*
> - *Beberrão* = estritamente abaixo de 8 km/l
> - *Super Econômico* = estritamente acima de 18 km/l