# 06 · Decisões e aprendizados

Esta parte é o "diário de bordo" — as decisões que moldaram o projeto e as armadilhas que só aparecem fazendo. É aqui que fica registrado o raciocínio, não só o resultado.

## Decisões que definiram o projeto

**Deixar o dado moldar a história.** A ideia inicial era um RFM na Página 4. Quando o profiling mostrou a base 1:1 (zero recompra), forçar o RFM teria sido teatro. Reposicionei a página para tratar a **compra única como o achado**. Analista bom ajusta a análise ao dado, não o contrário.

**Fixar a janela de análise cedo.** Descobrir os 2 registros de jan/2024 e filtrar para 2023 no Power Query (não por fórmula no Excel) garantiu que os dois ambientes reconciliassem em 998 transações. Se o filtro ficasse só no Excel, o Power BI continuaria com 1.000 e a validação acusaria uma divergência fantasma.

**Analisar por eliminação.** No achado central (o que dirige o alto valor), não cravei a primeira hipótese plausível. Testei categoria, testei idade, refutei as duas, e só então confirmei quantidade. As duas leituras refutadas são tão valiosas quanto a confirmada — mostram que a conclusão foi derivada, não chutada.

**Saber quando NÃO fazer algo.** Não criei uma dimensão de cliente (seria inútil numa base 1:1). Não fiz Pareto com `RANKX` (o tier de valor resolveu a leitura de concentração de forma mais simples). Não usei visuais customizados nem forçei "jornada do cliente" onde a base não tem esse conceito. Restrição é sinal de critério.

## Armadilhas do Power BI / Excel encontradas

Estas custaram tempo de depuração — e por isso valem registro:

**1. Data como texto (localidade).** CSV `dd/mm/yyyy` lido em ambiente pt-BR sem "Usando Localidade" faz dia > 12 virar texto, silenciosamente. Quebra toda a _time intelligence_. Solução: tipar a data com localidade pt-BR no Power Query.

**2. `ÚNICO()` retornando 1.** Contagem de distintos com `ÚNICO` sobre referência de `INDIRECT` colapsa por _implicit intersection_. Solução robusta: `SOMARPRODUTO(1/CONT.SE(intervalo; intervalo))`, que não depende de _dynamic arrays_.

**3. O relacionamento cai no _reload_.** Toda vez que editei o M da coluna de data (renomear, mudar tipo), o Power BI viu _drop + add_ da coluna e o **relacionamento fixado nela caiu** — sem erro. O sintoma é inconfundível: medida de tempo dando `total ÷ 1, ÷ 2, ÷ 3...` ou receita _flat_ em todos os meses, enquanto os cards continuam certos (cards não dependem do relacionamento). Isso aconteceu mais de uma vez até virar checklist.

**4. Três coisas caem juntas no _reload_.** Depois de qualquer edição no Power Query, três configurações voltam ao padrão em silêncio: **relacionamento ativo**, **tabela de datas marcada** e **_Sort by Column_** das colunas de mês/dia. Viraram um checklist pós-_reload_ obrigatório.

**5. Eixo secundário mentindo.** No combo de faixa etária, o eixo do ticket começava em 450, exagerando uma queda de ~15% para parecer despencar. Começar o eixo em **zero** revelou a verdade: o ticket varia pouco entre faixas.

**6. Ordenação por valor vs cronológica.** Gráfico de linha ordenado por valor de medida "sobe" e finge crescimento. Precisa de `NomeMesAbreviado` classificado por `NumMes` **e** o eixo do visual ordenado pela coluna certa (o _Sort by Column_ sozinho não basta — o visual mantém o próprio critério).

**7. Título do visual duplicando o do fundo.** Como o layout traz o título de cada zona assado na imagem, o título do próprio visual precisa ficar **desligado** — senão duplica.

## Design: estrutura, não decoração

O layout executivo (sidebar de navegação, zonas de gráfico com título, slots de KPI) foi feito em SVG/PNG como plano de fundo, com paleta petróleo/laranja. A filosofia foi **cor com significado**: petróleo é a base, laranja é o accent reservado só para o insight-chave (o sábado, o tier Alto), categoria com cor fixa nas quatro páginas. Nada de dar uma cor para cada gráfico — policromia lê como falta de critério.

## O que este projeto me ensinou

- **Profiling não é burocracia** — foi onde os três achados que redefiniram o projeto apareceram.
- **A honestidade é um ativo** — declarar as limitações (base sintética, 1:1, 1 ano) comunica mais maturidade do que escondê-las.
- **Fundamento > sofisticação** — um star schema limpo com _time intelligence_ correta impressiona mais do que técnica avançada mal amarrada.
- **A narrativa é a skill mais rara** — transformar "receita por categoria" em "a alavanca é quantidade por transação, não mix de produto" é o que separa quem monta gráfico de quem analisa.
