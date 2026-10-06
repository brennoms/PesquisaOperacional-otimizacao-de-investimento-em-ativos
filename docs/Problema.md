## 03 Problema 1 — Alocação de Carteira (Finanças)

Link para a origem do trabalho:
- 

### 3.1 enunciado

A Ápice Capital Gestora de Recursos Ltda. é uma gestora de recursos fictícia, responsável pela administração de uma carteira multimercado de R$ 10.000.000,00 (dez milhões de reais), pertencente a um fundo de pensão igualmente fictício. O comitê de investimentos da gestora solicitou um estudo quantitativo para definir a alocação desse capital entre quatro classes de ativos disponíveis no mercado brasileiro, respeitando um conjunto de políticas internas de risco e liquidez aprovadas pelo comitê de risco da gestora.

As quatro classes de ativos consideradas são: (a) fundos referenciados DI/CDI, de liquidez diária e baixo risco; (b) títulos públicos prefixados, cujo retorno esperado deve ser tomado como equivalente mensal da meta da taxa Selic média vigente no período; (c) títulos públicos indexados à inflação (Tesouro IPCA+), cujo retorno mensal deve ser composto pela variação média do IPCA somada a um spread médio de mercado de 6,00% ao ano, praticado em títulos de médio prazo dessa natureza; e (d) fundos cambiais, referenciados à variação da taxa de câmbio comercial do dólar americano.

A política de investimentos da gestora estabelece as seguintes regras, válidas para o período de alocação:

* no mínimo 15% do capital deve permanecer em fundos DI/CDI, por exigência de liquidez imediata;
* no máximo 40% do capital pode ser alocado em títulos prefixados, dado o risco de marcação a mercado dessa classe;
* no mínimo 10% do capital deve ser destinado a títulos indexados à inflação, como proteção do poder de compra do fundo;
* por política de hedge cambial, entre 5% e 15% do capital deve permanecer em fundos cambiais;
* o "risco ponderado" da carteira — a soma das frações alocadas em cada classe, multiplicadas pela volatilidade histórica média mensal (desvio-padrão) da respectiva classe — não pode exceder 0,35% ao mês.

**Em resumo:** a empresa deseja determinar o valor MÁXIMO do retorno mensal esperado da carteira, decidindo que fração do capital alocar em cada um dos quatro tipos de investimento, sem desrespeitar nenhuma das regras internas listadas acima.

### 3.2 dados — de onde vêm e como obtê-los

Os quatro indicadores usados neste problema (CDI, Selic, IPCA e dólar comercial) são publicados diariamente ou mensalmente pelo Banco Central do Brasil, de forma gratuita, no Sistema Gerenciador de Séries Temporais (SGS).

| Classe de ativo | Indicador | Link oficial (fonte original) |
| --- | --- | --- |
| DI/CDI | Série SGS 12 | api.bcb.gov.br/dados/serie/bcdata.sgs.12/dados |
| Prefixado | Série SGS 432 (meta Selic) | api.bcb.gov.br/dados/serie/bcdata.sgs.432/dados |
| IPCA+ | Série SGS 433 (IPCA mensal) | api.bcb.gov.br/dados/serie/bcdata.sgs.433/dados |
| Cambial | Série SGS 1 (dólar comercial) | api.bcb.gov.br/dados/serie/bcdata.sgs.1/dados |

Para não exigir que a turma acesse diretamente esses links (que retornam dados em formato de código, não em planilha), os quatro indicadores já foram baixados e organizados na planilha única `Problema1_Financas_DadosBCB.xlsx`, com uma aba para cada série (CDI_serie12, Selic_serie432, IPCA_serie433 e Dolar_serie1), cada uma com duas colunas: Data e Valor.

**Observação:** a mesma planilha bruta (Problema1_Financas_DadosBCB.xlsx) também está disponível na pasta de materiais da disciplina. Assim que essa pasta for publicada, o arquivo poderá ser baixado diretamente em [claytonjasilva.github.io/progLinear/trabalhoap1/Problema1_Financas_DadosBCB.xlsx](https://claytonjasilva.github.io/progLinear/trabalhoap1/Problema1_Financas_DadosBCB.xlsx) — essa é a forma mais simples de obter o arquivo, sem precisar acessar o site de origem.

**Passo a passo para obter e organizar os dados**

1. Baixe o arquivo Problema1_Financas_DadosBCB.xlsx (endereço indicado na observação acima) e abra-o no Excel.
2. Confira as quatro abas do arquivo. Cada uma traz a série completa, do dia 01/01/2023 até 01/09/2026, exatamente como publicada pelo Banco Central — nenhum tratamento foi feito ainda.
3. Verifique se há linhas em branco, datas repetidas ou valores ausentes em cada aba; se houver, remova-as antes de calcular qualquer coisa (limpeza dos dados).
4. Guarde o arquivo limpo: ele será a base para os cálculos da seção 3.3.

### 3.3 o que deve ser entregue

**i) Extração e limpeza dos dados.** Confirme que as quatro abas da planilha estão completas e sem linhas duplicadas ou em branco. Entenda o que cada coluna representa: nas abas CDI e Selic, "Valor" é uma taxa de juros (% ao dia e % ao ano, respectivamente); na aba IPCA, "Valor" já é a variação mensal do índice (%); na aba Dólar, "Valor" é o preço do dólar em reais naquele dia.

**ii) Modelagem com dados médios (fazer no Excel).** Em cada aba, crie uma coluna auxiliar ao lado dos valores e calcule os retornos e riscos abaixo. Depois de calculados, esses quatro pares (retorno, risco) são os únicos números que alimentam o modelo de Programação Linear.

* **CDI (taxa diária, % ao dia):**
Em uma coluna auxiliar, calcule, para cada linha, o equivalente mensal daquela taxa diária com a fórmula
`=((1+B2/100)^21-1)*100`
(o expoente 21 representa o número aproximado de dias úteis em um mês). Depois, use `=MÉDIA()` e `=DESVPAD.A()` sobre essa coluna auxiliar para obter o retorno médio mensal e o risco (volatilidade) do CDI.
* **Selic (taxa anual, % ao ano):**
Mesma ideia, mas a fórmula da coluna auxiliar é
`=((1+B2/100)^(1/12)-1)*100`
Em seguida, use MÉDIA() e DESVPAD.A() sobre essa coluna.
* **IPCA (já é mensal):**
Não precisa de coluna auxiliar. Use =MÉDIA() e =DESVPAD.A() diretamente sobre a coluna Valor. Para obter o retorno do IPCA+, some ao resultado da MÉDIA() o valor do spread mensal de 6% a.a., que se calcula uma única vez com
`=((1,06)^(1/12)-1)*100`
O risco do IPCA+ é o mesmo desvio-padrão do IPCA.
* **Dólar (preço diário, em R$):**
Crie uma coluna auxiliar com a variação percentual entre um dia e o dia anterior:
`=(B3-B2)/B2*100`
Calcule a MÉDIA() e o DESVPAD.A() dessa coluna (valores diários) e, em seguida, converta para a escala mensal multiplicando a média por 21 e o desvio-padrão por `=RAIZ(21)` (regra simples de conversão de escala do dia para o mês).

Com os 4 retornos e os 4 riscos em mãos, **monte o modelo de Programação Linear** definindo 4 variáveis de decisão (a fração do capital em cada classe) e escrevendo a função-objetivo e as restrições exatamente como descritas no enunciado (item 3.1). Resolva o modelo no Solver do Excel.

**iii) Interpretação da solução ótima.** Depois de rodar o Solver, veja quanto ficou alocado em cada classe. Para cada restrição do enunciado, diga se ela ficou "no limite" (ativa) ou "com sobra" (com folga) e explique, em suas próprias palavras, por que isso faz sentido para o negócio.

**iv) Análise de sensibilidade.** Gere o Relatório de Sensibilidade do Solver (na janela de resultados do Solver, marque a caixa "Sensibilidade" antes de clicar em OK) e responda às perguntas formuladas a seguir.

**v) Conclusão.** Escreva, em um parágrafo, qual alocação você recomendaria ao comitê de investimentos, quanto de retorno mensal (em reais) essa alocação gera, e se alguma das políticas internas está "custando" retorno à carteira.

---

### Análise de sensibilidade — conceito

**O que é a análise de sensibilidade?**

Depois de resolver um modelo de Programação Linear, é natural perguntar: "o que aconteceria se um dos números do problema mudasse um pouco?" A análise de sensibilidade responde exatamente a isso, sem precisar refazer o modelo do zero. Ela mostra, para cada restrição, um número chamado preço-sombra (ou valor dual): o quanto o resultado ótimo (aqui, o retorno mensal da carteira) mudaria se o limite daquela restrição aumentasse em uma unidade. Uma restrição que está "no limite" (ativa) quase sempre tem um preço-sombra diferente de zero — ela está de fato impedindo um resultado melhor. Já uma restrição "com sobra" (com folga) tem preço-sombra igual a zero — ela não está atrapalhando o resultado, então relaxá-la não ajudaria em nada. O Solver do Excel calcula esses valores automaticamente no Relatório de Sensibilidade, na coluna "Sombra Preço" (ou "Shadow Price").

Com base nesse conceito, responda no seu relatório:

* Qual é o preço-sombra da restrição de concentração máxima em Prefixado (40%)? O que ele significa em pontos percentuais de retorno?
* Qual é o preço-sombra da restrição de proteção mínima em IPCA+ (10%)? Quanto a carteira "deixa de ganhar" por mês por causa dessa exigência?
* Qual é o preço-sombra da restrição de hedge cambial mínimo (5%)? Compare esse valor com os demais: qual política interna é a mais "cara" para a rentabilidade da carteira?
* O orçamento de risco (0,35% ao mês) está ativo ou tem folga na solução ótima? Se o comitê decidisse reduzir esse limite, a partir de que valor a solução ótima começaria a mudar?