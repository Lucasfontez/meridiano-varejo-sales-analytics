# 03 · ETL e modelagem

## Filosofia: a transformação vive no Power Query

A regra de ouro do projeto: **tudo que alimenta o modelo mora no Power Query (M), nunca em célula digitada à mão.** Fórmula e pivot servem para exploração e validação; o pipeline que gera o dado do dashboard precisa ser reprodutível e auditável. Isso também é o que transfere direto para o Power BI — o mesmo M roda nos dois.

## O pipeline — `tRaw` → `tVendas`

A base bruta entra como `tRaw` (import fiel do CSV, intocado). A tabela do modelo, `tVendas`, é uma **referência** a partir dela — assim `tRaw` continua sendo a fonte da verdade, e o profiling pode mostrar os 13 meses reais enquanto o modelo trabalha só com 2023.

Código M final da `tVendas`:

```m
let
    Fonte = Csv.Document(
        File.Contents("...\02-dados\raw\retail_sales_dataset.csv"),
        [Delimiter=",", Columns=9, Encoding=65001, QuoteStyle=QuoteStyle.None]
    ),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Fonte, [PromoteAllScalars=true]),

    // tipos base — valores monetários como decimal (não inteiro, para não truncar centavo)
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos", {
        {"Transaction ID", Int64.Type}, {"Customer ID", type text}, {"Gender", type text},
        {"Age", Int64.Type}, {"Product Category", type text}, {"Quantity", Int64.Type},
        {"Price per Unit", type number}, {"Total Amount", type number}
    }),

    // Date com localidade pt-BR — parse dd/mm correto (sem isso, dia > 12 vira texto)
    #"Data com Localidade" = Table.TransformColumnTypes(#"Tipo Alterado", {{"Date", type date}}, "pt-BR"),

    #"Colunas Renomeadas" = Table.RenameColumns(#"Data com Localidade", {
        {"Transaction ID", "ID Transações"}, {"Date", "Data"}, {"Customer ID", "ID Cliente"},
        {"Gender", "Gênero"}, {"Age", "Idade"}, {"Product Category", "Categoria Produto"},
        {"Quantity", "Quantidade"}, {"Price per Unit", "Preço Unitário"}, {"Total Amount", "Valor Total"}
    }),

    // tradução na fonte — garante consistência de rótulo em todos os visuais
    NomeMasculinoBR = Table.ReplaceValue(#"Colunas Renomeadas", "Male", "Masculino", Replacer.ReplaceText, {"Gênero"}),
    NomeFemininoBR = Table.ReplaceValue(NomeMasculinoBR, "Female", "Feminino", Replacer.ReplaceText, {"Gênero"}),

    // janela de análise: 2023 (remove as 2 transações de jan/2024 → 998 linhas)
    Filtrado2023 = Table.SelectRows(NomeFemininoBR, each Date.Year([Data]) = 2023),

    // coluna derivada: faixa etária (banding condicional — idiomático no PQ)
    FaixaEtaria = Table.AddColumn(Filtrado2023, "Faixa Etária", each
        if [Idade] <= 25 then "18-25"
        else if [Idade] <= 35 then "26-35"
        else if [Idade] <= 50 then "36-50"
        else "51+", type text),

    NomeBCategoriaBR = Table.ReplaceValue(FaixaEtaria, "Beauty", "Beleza", Replacer.ReplaceText, {"Categoria Produto"}),
    NomeVCategoriaBR = Table.ReplaceValue(NomeBCategoriaBR, "Clothing", "Vestuário", Replacer.ReplaceText, {"Categoria Produto"}),
    NomeECategoriaBR = Table.ReplaceValue(NomeVCategoriaBR, "Electronics", "Eletrônicos", Replacer.ReplaceText, {"Categoria Produto"}),

    // coluna derivada: faixa de valor (tiers ancorados no histograma de Valor Total)
    FaixaValor = Table.AddColumn(NomeECategoriaBR, "Faixa de Valor", each
        if [Valor Total] < 150 then "1. Baixo"
        else if [Valor Total] < 600 then "2. Médio"
        else "3. Alto", type text),

    #"Colunas Reordenadas" = Table.ReorderColumns(FaixaValor, {
        "ID Transações", "Data", "ID Cliente", "Gênero", "Idade", "Faixa Etária",
        "Categoria Produto", "Quantidade", "Preço Unitário", "Valor Total", "Faixa de Valor"
    })
in
    #"Colunas Reordenadas"
```

Decisões embutidas no código:
- **Encoding UTF-8 (65001)** e **localidade pt-BR na data** — os dois pontos que mais quebram silenciosamente.
- **Valores monetários como decimal** (`type number`), não inteiro — inteiro trunca centavo sem avisar.
- **Tradução e _banding_ na fonte** — categoria/gênero traduzidos uma vez no PQ, então todos os visuais falam o mesmo idioma.
- **Prefixo numérico nas faixas** (`1. Baixo`, `2. Médio`, `3. Alto`) — resolve a ordenação sem precisar de _Sort by Column_.

Um detalhe que custou depuração: ao editar o M à mão, um passo (`FaixaValor`) ficou apontando para um ramo anterior da cadeia, virando um "galho morto" que o `in` não retornava. Lição: **cada passo do M referencia o passo imediatamente anterior, e o `in` tem que devolver o último** — a UI encadeia sozinha, mas colar no Editor Avançado não.

## A dimensão calendário — `dCalendario`

O Power Pivot do Excel **não suporta tabela calculada em DAX** (o `CALENDAR()`/`CALENDARAUTO()` que se usaria no Power BI não existe lá). Então a `dCalendario` nasceu no Power Query, via `List.Dates`:

```m
let
    Início = #date(2023,1,1),
    Fim = #date(2023,12,31),
    Dias = List.Dates(Início, Duration.Days(Fim - Início) + 1, #duration(1,0,0,0)),
    Tabela = Table.FromList(Dias, Splitter.SplitByNothing(), {"Data"}),
    Tipada = Table.TransformColumnTypes(Tabela, {{"Data", type date}}),
    // + colunas derivadas: Ano, NumMes, NomeMes, NomeMesAbreviado, MesAno,
    //   Trimestre, Dia da Semana, NomeDia (e chaves numéricas para ordenação)
    ...
in
    Resultado
```

Cobrindo exatamente a janela de 2023 — por isso o filtro dos 2 registros de jan/2024 foi importante: se tivessem ficado, haveria data órfã sem correspondência no calendário.

## O modelo — star schema

Um esquema em estrela enxuto, honesto para uma base 1:1:

- **`fVendas`** — a fato (grão = transação). Como a base é 1:1, a demografia (Gênero, Idade, Faixa Etária) ficou como **atributo na própria fato** — uma dimensão de cliente teria a mesma cardinalidade da fato, seria inútil.
- **`dCalendario`** — a única dimensão que se paga, porque _time intelligence_ exige uma tabela de datas marcada.
- Relacionamento **`fVendas[Data]` → `dCalendario[Data]`**, cardinalidade **`* : 1`**, direção única.
- `dCalendario` **marcada como Tabela de Datas** (sem isso, `TOTALYTD`/`DATEADD`/`DATESINPERIOD` degradam em silêncio).
- **_Sort by Column_** aplicado: `NomeMesAbreviado` por `NumMes` e `NomeDia` por `Dia Semana Nº` — para ordenar cronologicamente, não em ordem alfabética.

Saber **quando não criar** uma dimensão (a de cliente, aqui) foi tão importante quanto saber montar o star schema.

## Excel primeiro, depois Power BI

Todo o modelo foi montado e as medidas escritas **primeiro no Power Pivot do Excel**, depois replicados no Power BI. Como as engines são as mesmas, foi treino com transferência quase total — só mudou o separador de argumentos (`;` no Excel, `,` no Power BI) e o _parse_ de data no import. Ao final, os números batem nos dois ambientes: essa reconciliação é a "fonte única da verdade" do projeto.
