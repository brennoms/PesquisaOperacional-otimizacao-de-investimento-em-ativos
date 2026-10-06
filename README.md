# Otimização de Alocação de Carteira (Programação Linear)

Este repositório contém a modelagem, os dados e os resultados do **Problema 1: Alocação de Carteira (Finanças)**, focado em maximizar o retorno esperado de um fundo de pensão fictício (Ápice Capital), respeitando rigorosas políticas internas de liquidez, hedge e orçamento de risco.

O projeto aplica conceitos de Pesquisa Operacional e otimização linear para a tomada de decisão no mercado financeiro, desenvolvido como parte das atividades acadêmicas do curso de Ciência de Dados e Inteligência Artificial (Ibmec).

* Link para a origem do trabalho: [claytonjasilva.github.io/progLinear/trabalhoap1/trabalho-ap1-aluno.html](https://claytonjasilva.github.io/progLinear/trabalhoap1/trabalho-ap1-aluno.html)

* Tutor: Clayton Silva [@claytonjasilva](https://github.com/claytonjasilva)

---

## 🎯 Objetivo do Projeto

Determinar a alocação ótima de um capital de R$ 10.000.000,00 distribuído entre quatro classes de ativos brasileiros:

1. **Fundos DI/CDI** (Baixo risco, alta liquidez)
2. **Títulos Prefixados** (Risco de marcação a mercado, atrelados à Selic)
3. **Tesouro IPCA+** (Proteção inflacionária + spread)
4. **Fundos Cambiais** (Hedge atrelado ao Dólar Comercial)

O modelo busca o **valor máximo do retorno mensal esperado**, sujeito às seguintes restrições:

* Mínimo de 15% em DI/CDI.
* Máximo de 40% em Prefixados.
* Mínimo de 10% em IPCA+.
* Entre 5% e 15% em Fundos Cambiais.
* Risco ponderado máximo da carteira de 0,35% ao mês.

---

## 📂 Estrutura do Repositório

* `dados/`: Contém a base bruta **Problema1_Financas_DadosBCB.xlsx** extraída, pelo tutor, do Sistema Gerenciador de Séries Temporais (SGS) do Banco Central do Brasil.
* `modelo/`: Planilhas contendo a limpeza dos dados, o cálculo dos parâmetros (retorno médio e desvio-padrão/risco) e a modelagem via Solver.
* `relatorio/`: Documentação final com a interpretação da solução ótima, alocação sugerida e análise de sensibilidade.
* `apresentacao/`: Slides e documentos relacionados a apresentação do trabalho.
* `src/`: Scripts de Extração e preparação.

---

## ⚙️ Metodologia e Etapas

1. **Coleta e Limpeza de Dados:**
* Extração de séries históricas (SGS 12, 432, 433, 1) compreendendo o período de 01/01/2023 a 01/09/2026.
* Tratamento de valores ausentes e inconsistências nas datas.


2. **Engenharia de Parâmetros:**
* Conversão de taxas diárias e anuais para escalas mensais.
* Cálculo de média (retorno esperado) e desvio-padrão populacional (risco) para cada classe de ativo.


3. **Modelagem Matemática:**
* Definição de 4 variáveis de decisão contínuas (fração do capital).
* Estruturação da Função Objetivo (Maximizar retorno) e Restrições de negócio.
* Resolução algorítmica utilizando o Solver.


4. **Análise de Sensibilidade (Pós-otimização):**
* Avaliação dos preços-sombra (valor dual) para entender o custo de oportunidade de cada política interna (restrições ativas vs. com folga).



---

## 🛠️ Ferramentas Utilizadas

* **Extração de Dados:** Banco Central do Brasil (SGS API), pandas.
* **Tratamento e Modelagem:** Microsoft Excel (Fórmulas estatísticas e matemáticas)
* **Otimização:** Excel Solver (Motor de Programação Linear Simplex LP)
