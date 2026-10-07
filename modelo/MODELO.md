# Problema 1 — Modelo de Programação Linear

**Alocação ótima de carteira · Ápice Capital Gestora de Recursos Ltda.**

Documento do modelo matemático: variáveis, função-objetivo, restrições, solução do Solver,
relatório de sensibilidade e interpretações.

Planilha correspondente: `modelo/P1_Modelo_Carteira.xlsx`
Base de dados: `dados/Problema1_Financas_DadosBCB_20261006_123832.xlsx`
Período coberto: 01/01/2023 a 01/09/2026

---

## 1. Resumo da solução

| | |
|---|---|
| **Alocação ótima** | DI/CDI 45% · Prefixado 40% · IPCA+ 10% · Cambial 5% |
| **Em reais** | R$ 4.500.000 · R$ 4.000.000 · R$ 1.000.000 · R$ 500.000 |
| **Retorno esperado** | **0,963008% ao mês** |
| **Retorno em reais** | **R$ 96.300,78 por mês** |
| **Risco ponderado** | 0,287470% ao mês (limite: 0,35%) |
| **Política mais cara** | Hedge cambial mínimo — custa R$ 1.062,43/mês por ponto percentual |

---

## 2. Parâmetros estimados a partir dos dados

Os oito números abaixo são os únicos que alimentam o modelo. Todos em **% ao mês**.

| Variável | Classe de ativo | Retorno médio (%) | Risco — desvio-padrão (%) | Série de origem |
|---|---|---|---|---|
| x₁ | Fundos DI/CDI | 1,029889 | 0,120000 | SGS 12 |
| x₂ | Títulos prefixados | 1,037410 | 0,119448 | SGS 432 |
| x₃ | Tesouro IPCA+ | 0,862210 | 0,296722 | SGS 433 + spread |
| x₄ | Fundos cambiais | **−0,032545** | **3,120366** | SGS 1 |

### Como cada parâmetro foi obtido

**DI/CDI** — a série traz a taxa em % ao dia. Cada observação foi convertida ao equivalente
mensal por `=((1+Valor/100)^21-1)*100`, com 21 dias úteis por mês. Média e `DESVPAD.A` sobre
a coluna auxiliar.

**Prefixado** — a série traz a meta Selic em % ao ano. Conversão ao equivalente mensal por
`=((1+Valor/100)^(1/12)-1)*100`. Média e `DESVPAD.A` sobre a coluna auxiliar.

**IPCA+** — a série já é mensal, dispensando coluna auxiliar. O retorno é a média do IPCA
(0,375455%) somada ao spread mensal equivalente a 6% a.a., calculado uma única vez por
`=((1,06)^(1/12)-1)*100` = 0,486755%. O risco é o desvio-padrão do próprio IPCA.

**Cambial** — a série traz o preço do dólar em reais. Foi criada a coluna de variação
percentual diária, `=(B3-B2)/B2*100`; a média diária (−0,001550%) foi multiplicada por 21 e o
desvio-padrão diário (0,680920%) por `=RAIZ(21)`, conforme a regra de conversão de escala
indicada no enunciado.

### Nota sobre o desvio-padrão

A função `DESVPAD.A` (`STDEV.S`) calcula o desvio-padrão **amostral**, com denominador *n*−1.
É a função indicada no enunciado e a adotada aqui.

### Nota sobre a limpeza dos dados (item i)

As quatro abas foram conferidas e **não apresentaram** datas duplicadas, valores ausentes ou
linhas em branco. Nenhuma remoção foi necessária; o arquivo limpo é idêntico ao bruto.

As séries têm tamanhos diferentes, o que **não é inconsistência** e sim reflexo da frequência
de publicação de cada indicador:

| Série | Linhas | Frequência |
|---|---|---|
| CDI | 921 | dias úteis |
| Selic (meta) | 1.340 | dias corridos — a meta vigora todos os dias |
| IPCA | 44 | mensal |
| Dólar | 921 | dias úteis |

---

## 3. Modelo matemático

### Variáveis de decisão

Fração do capital de R$ 10.000.000,00 alocada em cada classe:

```
x₁ = fração em fundos DI/CDI
x₂ = fração em títulos prefixados
x₃ = fração em Tesouro IPCA+
x₄ = fração em fundos cambiais
```

A escolha de trabalhar com **frações**, e não com valores em reais, mantém os coeficientes na
mesma ordem de grandeza e permite ler a solução diretamente como política de alocação. A
conversão para reais é feita ao final, multiplicando por R$ 10 milhões.

### Função-objetivo

Maximizar o retorno mensal esperado da carteira, em % ao mês:

```
Máx Z = 1,029889·x₁ + 1,037410·x₂ + 0,862210·x₃ − 0,032545·x₄
```

### Restrições

```
(R1)  x₁ + x₂ + x₃ + x₄ = 1                                       alocação integral
(R2)  x₁                               ≥ 0,15                      liquidez imediata
(R3)         x₂                        ≤ 0,40                      concentração máxima
(R4)                x₃                 ≥ 0,10                      proteção inflacionária
(R5)                       x₄          ≥ 0,05                      hedge cambial mínimo
(R6)                       x₄          ≤ 0,15                      hedge cambial máximo
(R7)  0,120000·x₁ + 0,119448·x₂ + 0,296722·x₃ + 3,120366·x₄ ≤ 0,35  orçamento de risco
      x₁, x₂, x₃, x₄ ≥ 0                                           não negatividade
```

### Por que R1 existe, mesmo não estando na lista de políticas

O enunciado descreve **cinco** políticas internas, mas o modelo exige **sete** restrições. A
restrição R1 não aparece na lista porque é uma condição implícita do problema: trata-se de
alocar *todo* o capital sob gestão.

Sem R1, o modelo é formalmente válido e todas as cinco políticas continuam satisfeitas, mas o
Solver pode devolver uma carteira que investe apenas parte do capital — deixando recursos sem
aplicação, o que não corresponde ao mandato de uma gestora. R1 é o que torna x₁ a x₄ frações
de um todo em vez de quantidades independentes.

### Verificação de unidades

Retornos, riscos e o orçamento de risco estão todos em **pontos percentuais ao mês**. O limite
de R7 é 0,35 — e não 0,0035 — na mesma escala dos coeficientes σ. Misturar as duas escalas
torna o modelo inviável ou produz uma solução sem sentido econômico.

### Forma padrão

Com variáveis de folga (f), excesso (e) e artificiais (a), para aplicação do Simplex:

```
x₁ + x₂ + x₃ + x₄                                      + a₁ = 1
x₁                                            − e₂      + a₂ = 0,15
       x₂                              + f₃                  = 0,40
              x₃                              − e₄      + a₄ = 0,10
                     x₄                       − e₅      + a₅ = 0,05
                     x₄                + f₆                  = 0,15
0,120000x₁ + 0,119448x₂ + 0,296722x₃ + 3,120366x₄ + f₇       = 0,35
```

Todas as variáveis ≥ 0. As restrições de tipo ≥ e a igualdade não admitem base inicial pronta,
exigindo variáveis artificiais e o método das duas fases (ou Big-M). O Solver do Excel executa
esse tratamento internamente ao se escolher o método **LP Simplex**.

---

## 4. Solução ótima

Configuração do Solver: objetivo em Z, **Máx**, células variáveis x₁ a x₄, as sete restrições
acima, opção "tornar variáveis irrestritas não negativas" marcada, método **LP Simplex**.

| Classe | Fração | Valor alocado |
|---|---|---|
| DI/CDI | 45,00% | R$ 4.500.000,00 |
| Prefixado | 40,00% | R$ 4.000.000,00 |
| IPCA+ | 10,00% | R$ 1.000.000,00 |
| Cambial | 5,00% | R$ 500.000,00 |
| **Total** | **100,00%** | **R$ 10.000.000,00** |

**Z = 0,963008% ao mês**, equivalente a **R$ 96.300,78 por mês**.

---

## 5. Interpretação da solução ótima (item iii)

| Restrição | Situação | Folga | Leitura de negócio |
|---|---|---|---|
| R1 — alocação integral | ativa | — | igualdade, sempre ativa por construção |
| R2 — liquidez ≥ 15% | **com folga** | 30,0 p.p. | não limita; o CDI recebe 45% por ser residual |
| R3 — concentração ≤ 40% | **ativa** | 0 | trava real: o prefixado é o ativo mais rentável |
| R4 — proteção ≥ 10% | **ativa** | 0 | obriga posição em classe menos rentável |
| R5 — hedge ≥ 5% | **ativa** | 0 | obriga posição em classe de retorno negativo |
| R6 — hedge ≤ 15% | **com folga** | 10,0 p.p. | o modelo foge do câmbio; o teto nunca é atingido |
| R7 — risco ≤ 0,35% | **com folga** | 0,0625 | usa 0,287470 dos 0,35 disponíveis |

### Por que o prefixado trava no teto

É o ativo de maior retorno (1,037410%) e, simultaneamente, o de menor risco (0,119448%). Não há
razão para não maximizá-lo. A política de concentração de 40% é a única coisa que o impede.

### Por que a liquidez sobra tanto

O CDI recebe 45% — três vezes o mínimo exigido — mas isso **não decorre da política de
liquidez**. Com o prefixado travado em 40% e IPCA+ e câmbio nos respectivos pisos, o CDI é o
único destino possível para o capital restante. A política de liquidez não está, na prática,
influenciando a decisão: ela seria satisfeita de qualquer maneira.

### Por que o orçamento de risco sobra

À primeira vista é contraintuitivo que a restrição de risco não aperte. A explicação está na
composição do risco:

| Classe | Contribuição ao risco | % do risco total |
|---|---|---|
| DI/CDI | 0,054000 | 18,8% |
| Prefixado | 0,047779 | 16,6% |
| IPCA+ | 0,029672 | 10,3% |
| **Cambial** | **0,156018** | **54,3%** |
| **Total** | **0,287470** | 100% |

Os fundos cambiais respondem por **mais da metade do risco da carteira ocupando apenas 5% do
capital** — sua volatilidade (3,12%) é 26 vezes a do CDI. Como a política de hedge já limita
essa classe a 15%, ela acaba protegendo indiretamente o orçamento de risco antes que ele seja
atingido. As duas políticas se sobrepõem.

---

## 6. Análise de sensibilidade (item iv)

Preços-sombra extraídos do Relatório de Sensibilidade do Solver, expressos como variação de Z
por **1 ponto percentual** de afrouxamento do limite.

| Política | Preço-sombra (p.p. de Z) | Em reais por mês |
|---|---|---|
| R3 — concentração máxima em prefixado (40%) | **+0,0000752** | **+R$ 7,52** |
| R4 — proteção mínima em IPCA+ (10%) | **−0,0016768** | **−R$ 167,68** |
| R5 — hedge cambial mínimo (5%) | **−0,0106243** | **−R$ 1.062,43** |
| R2, R6, R7 — restrições com folga | 0 | R$ 0,00 |

### a) Preço-sombra da concentração máxima em prefixado

**+0,0000752 p.p. por ponto percentual** de teto adicional, ou R$ 7,52 por mês.

Elevar o teto de 40% para 41% permite migrar 1 p.p. do CDI para o prefixado, cuja vantagem de
retorno é de apenas 0,007521 p.p. (1,037410 − 1,029889). O ganho é real mas pequeno: as duas
classes são quase idênticas em retorno.

**Intervalo de validade:** o preço-sombra vale até o teto de **70%**. Nesse ponto o CDI atinge
seu piso de 15% e não há mais de onde migrar capital; acima de 70% o preço-sombra cai a zero e
Z estaciona em 0,965264%.

### b) Preço-sombra da proteção mínima em IPCA+

**−0,0016768 p.p. por ponto percentual** de piso, ou **R$ 167,68 por mês**.

Cada ponto percentual obrigatório em IPCA+ desloca capital do CDI (1,029889%) para uma classe
que rende 0,862210% — diferença de 0,167679 p.p. por unidade de fração. A exigência de 10%
custa, portanto, cerca de **R$ 1.676,79 por mês** em relação a uma carteira sem essa política.

### c) Preço-sombra do hedge cambial mínimo

**−0,0106243 p.p. por ponto percentual** de piso, ou **R$ 1.062,43 por mês**.

É a política **mais cara da carteira**, por uma ordem de grandeza: custa 6,3 vezes mais que a
proteção inflacionária e 141 vezes mais que o ganho de afrouxar a concentração. A exigência
total de 5% custa cerca de **R$ 5.312,17 por mês**.

A razão é que o dólar apresentou **retorno médio negativo** no período (−0,032545% ao mês):
cada real alocado em câmbio não apenas deixa de render o CDI como subtrai valor. A diferença
por unidade de fração é 1,029889 − (−0,032545) = 1,062434 p.p.

### d) O orçamento de risco está ativo ou com folga?

**Com folga.** A carteira ótima consome 0,287470% dos 0,35% autorizados — sobram 0,062530 p.p.
Seu preço-sombra é zero: ampliar o orçamento de risco não traria retorno algum.

**A partir de que valor a solução começaria a mudar?** Aqui o comportamento é peculiar e vale
destacar: a solução ótima **não muda gradualmente**. Testando valores decrescentes do limite:

| Limite de risco | Resultado |
|---|---|
| 0,350% | Z = 0,963008% (solução original) |
| 0,300% | Z = 0,963008% (inalterada) |
| 0,287470% | Z = 0,963008% (inalterada) |
| 0,287469% | **Inviável** |

O motivo é que a carteira de **retorno máximo coincide exatamente com a carteira de risco
mínimo** admissível pelas demais políticas. Não existe nenhuma alocação viável com risco
inferior a 0,287470%. Reduzir o orçamento abaixo desse valor não produz uma solução pior — produz
**ausência de solução**.

---

## 7. Observações críticas sobre o modelo

Três pontos que o modelo revela e que merecem registro.

### 7.1 Não há trade-off entre risco e retorno nestes dados

A premissa usual em finanças é que maior retorno exige maior risco. **Neste conjunto de dados
isso não ocorre.** As duas classes mais rentáveis (prefixado 1,0374% e CDI 1,0299%) são também
as menos voláteis (0,1194% e 0,1200%), enquanto a menos rentável (câmbio, −0,0325%) é de longe
a mais volátil (3,1204%).

A consequência é que o problema **degenera**: a carteira de maior retorno é idêntica à de menor
risco. O modelo não está arbitrando risco contra retorno — está apenas fugindo do dólar até o
limite que a política permite. Isso explica tanto a folga no orçamento de risco quanto o fato de
todas as restrições ativas serem pisos obrigatórios, e não tetos de recurso.

### 7.2 Duas políticas estão próximas de entrar em conflito

O hedge cambial mínimo e o orçamento de risco são compatíveis hoje, mas por margem estreita.
Elevando-se o piso de hedge:

| Hedge mínimo | Resultado |
|---|---|
| 5,00% | Z = 0,963008% |
| 6,00% | Z = 0,952383% |
| 7,00% | Z = 0,941759% |
| 7,08% | Z = 0,940909% |
| 7,09% | **Inviável** |

Acima de **7,08%** não existe carteira que satisfaça simultaneamente o hedge mínimo e o
orçamento de risco de 0,35%. Como os fundos cambiais sozinhos consomem 3,12 pontos de risco por
unidade de fração, elevar o hedge para os 15% que a própria política autoriza como teto tornaria
o modelo **inviável**. As duas políticas, escritas de forma independente pelo comitê, só
coexistem dentro de uma faixa estreita.

### 7.3 A medida de risco é uma simplificação

A restrição R7 soma as volatilidades individuais ponderadas pelas frações:

```
σ_carteira ≈ Σ σᵢ · xᵢ
```

Essa **não é** a volatilidade de uma carteira. A formulação correta seria

```
σ_carteira = √(xᵀ Σ x)
```

onde Σ é a matriz de covariâncias. A diferença é a **correlação** entre as classes: ativos que
não se movem juntos reduzem o risco agregado, efeito que a soma linear ignora. A aproximação
adotada portanto **superestima** o risco de uma carteira diversificada, e é conservadora nesse
sentido.

A simplificação é necessária: a formulação correta é quadrática e sairia do escopo da
Programação Linear, exigindo programação quadrática. O registro aqui é para deixar claro que o
limite de 0,35% deve ser lido como um orçamento de risco *no sentido definido pela política
interna*, e não como a volatilidade efetiva da carteira.

---

## 8. Conclusão e recomendação (item v)

Recomenda-se ao comitê de investimentos a alocação de **45% em fundos DI/CDI (R$ 4.500.000),
40% em títulos prefixados (R$ 4.000.000), 10% em Tesouro IPCA+ (R$ 1.000.000) e 5% em fundos
cambiais (R$ 500.000)**, que maximiza o retorno mensal esperado em **0,963008%, ou R$ 96.300,78
por mês**, respeitando integralmente as políticas internas vigentes.

Três políticas estão de fato limitando o resultado. A **exigência de hedge cambial mínimo é a
mais onerosa**, custando R$ 1.062,43 por mês para cada ponto percentual exigido — R$ 5.312,17
por mês no total — porque o dólar apresentou retorno médio negativo no período analisado. A
**proteção inflacionária mínima** custa R$ 167,68 por ponto percentual, ou R$ 1.676,79 no total.
O **teto de concentração em prefixados** é a única política cujo afrouxamento traria ganho, ainda
que modesto: R$ 7,52 por ponto percentual adicional, até o limite de 70%.

Já a **exigência de liquidez e o orçamento de risco não estão custando nada** à carteira: a
primeira é satisfeita com folga de 30 pontos percentuais por consequência das demais restrições,
e o segundo opera a 0,287% contra 0,35% autorizados.

Cabe registrar, contudo, que **a recomendação de reduzir o hedge cambial deve ser lida com
cautela**. O modelo avalia o câmbio exclusivamente por seu retorno histórico no período
2023–2026, em que o real se valorizou. A função do hedge em uma carteira de fundo de pensão não
é gerar retorno, e sim proteger contra cenários de desvalorização cambial — proteção cujo valor
não aparece em um modelo determinístico alimentado por médias históricas. O custo de
R$ 5.312,17 por mês deve ser entendido como **o prêmio pago pelo seguro**, não como ineficiência
a ser eliminada.

Recomenda-se ainda que o comitê **revise a compatibilidade entre o piso de hedge e o orçamento
de risco**: as duas políticas tornam-se mutuamente inviáveis acima de 7,08% de alocação cambial,
bem abaixo dos 15% que a própria política de hedge admite como teto.

---

## 9. Reprodução

1. Abrir `modelo/P1_Modelo_Carteira.xlsx`.
2. Abas `1_CDI` a `4_Dolar`: séries brutas com as colunas auxiliares em fórmula. Nenhum valor
   foi digitado como constante — todos os parâmetros recalculam a partir dos dados.
3. Aba `5_Parametros`: consolida os oito parâmetros por referência às abas anteriores.
4. Aba `6_Modelo`: variáveis em amarelo (zeradas), função-objetivo, as sete restrições e o
   roteiro de configuração do Solver.
5. Dados → Solver → Resolver, marcando **Resposta** e **Sensibilidade** antes do OK.
6. Conferir contra os valores deste documento.
