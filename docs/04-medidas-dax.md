# 04 · Medidas DAX

As 13 medidas do modelo, agrupadas por função. O código está na sintaxe do **Power BI** (separador `,`); no Excel Power Pivot, onde foram ensaiadas primeiro, o separador é `;`.

Todas ficam numa tabela dedicada `_Medidas` (sem relacionamento, só contêiner) — mantém o modelo organizado e é o mesmo padrão que se usa no Power BI.

Referências de coluna usadas: `fVendas[Valor Total]`, `fVendas[Quantidade]`, `fVendas[ID Cliente]`, `dCalendario[Data]`, `dCalendario[MesAno]`.

---

## Base

### `Receita Total`
```dax
Receita Total = SUM ( fVendas[Valor Total] )
```
Soma o valor de todas as transações no contexto de filtro atual. É a medida-âncora — quase todas as outras derivam dela. **Função:** `SUM`.

### `Itens Vendidos`
```dax
Itens Vendidos = SUM ( fVendas[Quantidade] )
```
Soma as unidades vendidas. Alimenta o preço médio ponderado. **Função:** `SUM`.

### `Nº Transações`
```dax
Nº Transações = COUNTROWS ( fVendas )
```
Conta as linhas da fato — como o grão é uma transação por linha, isso é o número de transações. Usei `COUNTROWS` em vez de contar uma coluna específica por ser mais direto e não depender de nenhuma coluna em particular. **Função:** `COUNTROWS`.

### `Nº Clientes`
```dax
Nº Clientes = DISTINCTCOUNT ( fVendas[ID Cliente] )
```
Conta clientes únicos. Numa base 1:1, o resultado é **igual** a `Nº Transações` (998) — e essa igualdade não é redundância, é a **prova no modelo** da tese de compra única. **Função:** `DISTINCTCOUNT`.

### `Ticket Médio`
```dax
Ticket Médio = DIVIDE ( [Receita Total], [Nº Transações] )
```
Valor médio por transação (AOV). `DIVIDE` trata divisão por zero de forma segura (retorna vazio em vez de erro). **Função:** `DIVIDE`.

### `Preço Médio`
```dax
Preço Médio = DIVIDE ( [Receita Total], [Itens Vendidos] )
```
Preço médio **ponderado por volume** — diferente de `AVERAGE(Preço Unitário)`, que é a média simples. A ponderada reflete o preço efetivo de fato: um item caro vendido em grande quantidade pesa mais. Saber a diferença entre as duas evita um erro comum de interpretação. **Função:** `DIVIDE`.

---

## Tempo (_time intelligence_)

> Todas exigem a `dCalendario` marcada como Tabela de Datas e o relacionamento ativo. Sem isso, retornam o total repetido em cada período — em silêncio.

### `Receita Acumulada`
```dax
Receita Acumulada = TOTALYTD ( [Receita Total], dCalendario[Data] )
```
Receita acumulada do início do ano até a data do contexto (_running total_). Como a base é de um ano, sobe de forma monotônica de zero até o total (R$ 454.470 em dezembro). A quase-linearidade dela é, por si só, um achado: faturamento constante, sem sazonalidade. **Função:** `TOTALYTD`.

### `Receita Mês Anterior`
```dax
Receita Mês Anterior = CALCULATE ( [Receita Total], DATEADD ( dCalendario[Data], -1, MONTH ) )
```
Receita do mês imediatamente anterior. É uma medida **auxiliar** — existe só para alimentar o MoM, então fica **oculta** no relatório (não é KPI). **Funções:** `CALCULATE`, `DATEADD`.

### `Receita MoM %`
```dax
Receita MoM % = DIVIDE ( [Receita Total] - [Receita Mês Anterior], [Receita Mês Anterior] )
```
Variação percentual da receita mês a mês. Janeiro fica em branco (não há dezembro anterior dentro da janela de 2023) — comportamento correto, não erro. **Função:** `DIVIDE`.

### `Média Móvel 3M`
```dax
Média Móvel 3M =
VAR Janela = DATESINPERIOD ( dCalendario[Data], MAX ( dCalendario[Data] ), -3, MONTH )
RETURN
    DIVIDE (
        CALCULATE ( [Receita Total], Janela ),
        CALCULATE ( DISTINCTCOUNT ( dCalendario[MesAno] ), Janela )
    )
```
Média móvel de 3 meses da receita — usada para suavizar o ruído (com ~83 transações/mês, o mês a mês oscila muito). O ponto não óbvio está no **denominador**: em vez de dividir fixo por 3, ele divide pela **quantidade de meses realmente presentes na janela** (`DISTINCTCOUNT` de `MesAno`). Isso evita inflar os dois primeiros meses, onde só há 1–2 meses de histórico. **Funções:** `VAR`/`RETURN`, `DATESINPERIOD`, `CALCULATE`, `DISTINCTCOUNT`, `DIVIDE`, `MAX`.

---

## Cliente e valor

### `Taxa de Recompra`
```dax
Taxa de Recompra =
DIVIDE (
    COALESCE (
        COUNTROWS ( FILTER ( VALUES ( fVendas[ID Cliente] ), [Nº Transações] > 1 ) ),
        0
    ),
    [Nº Clientes]
)
```
Percentual de clientes com mais de uma transação. O `FILTER` sobre `VALUES(ID Cliente)` mantém só os clientes com `Nº Transações > 1`; `COUNTROWS` os conta. O detalhe é o **`COALESCE(..., 0)`**: sem ele, quando não há nenhum cliente com recompra, a expressão retorna `BLANK` e o card fica **vazio, parecendo medida quebrada**. Forçando 0, o card exibe **"0%"** — que é a manchete da tese de compra única. Nesta base, retorna 0%. **Funções:** `DIVIDE`, `COALESCE`, `COUNTROWS`, `FILTER`, `VALUES`.

### `% Compra Única`
```dax
% Compra Única = 1 - [Taxa de Recompra]
```
Complemento da recompra — percentual de clientes que compraram uma única vez. Retorna **100%**. Junto com a Taxa de Recompra, forma o par de cards que crava a tese na Página 4. **Função:** aritmética simples sobre medida.

### `% Receita`
```dax
% Receita = DIVIDE ( [Receita Total], CALCULATE ( [Receita Total], ALLSELECTED ( fVendas ) ) )
```
Participação (_share_) da receita de um recorte sobre o total visível. `ALLSELECTED` remove o contexto do visual, mas **respeita os slicers da página** — então cada célula mostra sua fatia do total filtrado, não do total absoluto. Usada nos heatmaps (faixa etária × categoria) e na matriz de faixa de valor, para leitura de proporção. **Funções:** `DIVIDE`, `CALCULATE`, `ALLSELECTED`.

---

## Resumo

| # | Medida | Tipo | Função principal |
|---|---|---|---|
| 1 | Receita Total | Base | `SUM` |
| 2 | Itens Vendidos | Base | `SUM` |
| 3 | Nº Transações | Base | `COUNTROWS` |
| 4 | Nº Clientes | Base | `DISTINCTCOUNT` |
| 5 | Ticket Médio | Base | `DIVIDE` |
| 6 | Preço Médio | Base | `DIVIDE` (ponderado) |
| 7 | Receita Acumulada | Tempo | `TOTALYTD` |
| 8 | Receita Mês Anterior | Tempo (aux) | `DATEADD` |
| 9 | Receita MoM % | Tempo | `DIVIDE` |
| 10 | Média Móvel 3M | Tempo | `DATESINPERIOD` + denominador dinâmico |
| 11 | Taxa de Recompra | Cliente | `FILTER` + `COALESCE` |
| 12 | % Compra Única | Cliente | aritmética sobre medida |
| 13 | % Receita | Valor | `ALLSELECTED` |
