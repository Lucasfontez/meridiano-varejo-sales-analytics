# 02 · Dados e profiling

## O dataset

[Retail Sales Dataset](https://www.kaggle.com/datasets/mohammadtalib786/retail-sales-dataset) (Kaggle, por _mohammadtalib786_). Base transacional, ~1.000 linhas, referente a 2023. Grão: **uma linha = uma transação**.

### Dicionário de dados

| Coluna (original) | Renomeada (PT) | Descrição |
|---|---|---|
| Transaction ID | ID Transações | Identificador único da transação |
| Date | Data | Data da transação |
| Customer ID | ID Cliente | Identificador do cliente |
| Gender | Gênero | Male/Female → Masculino/Feminino |
| Age | Idade | Idade do cliente |
| Product Category | Categoria Produto | Electronics/Clothing/Beauty → Eletrônicos/Vestuário/Beleza |
| Quantity | Quantidade | Unidades na transação |
| Price per Unit | Preço Unitário | Preço por unidade |
| Total Amount | Valor Total | Valor total da transação |

## O profiling — e por que ele foi decisivo

Fiz o diagnóstico da base no Excel, importando via Power Query (tabela `tRaw`). O profiling não foi burocracia: **três achados aqui redefiniram o projeto inteiro.**

### Achado 1 — A data veio como texto (e isso é uma armadilha silenciosa)

O CSV traz a data em formato `dd/mm/yyyy`. Ao importar num ambiente pt-BR sem cuidado, o Power Query tenta interpretar como `mm/dd` (en-US) e **qualquer data com dia > 12 vira texto** — sem erro visível. Fórmula de data sobre texto retorna lixo.

A correção foi tipar a coluna com **"Usando Localidade" → Data → Português (Brasil)**, forçando o _parse_ correto. Detalhe pequeno, impacto enorme: sem isso, toda a análise temporal (sazonalidade, acumulada, média móvel) estaria errada.

### Achado 2 — Base 1:1: zero recompra

Ao contar clientes distintos contra transações, o resultado foi **998 clientes para 998 transações** — cada cliente comprou exatamente uma vez. A taxa de recompra é **0%**.

Isso matou a ideia original de fazer um RFM (recência/frequência/valor): não há recorrência para medir. Em vez de forçar uma análise que a base não sustenta, **transformei a compra única no próprio achado** — a Página 4 do dashboard foi reposicionada para provar essa tese. Deixar o dado moldar a história (e não o contrário) foi uma das decisões mais importantes do projeto.

### Achado 3 — A base vazava para 2024

Contando os meses distintos, deu **13, não 12**. Investigando: a base tinha **2 transações datadas de 01/01/2024** (IDs 211 e 650). Um 13º mês com apenas 2 registros arruinaria qualquer visual de sazonalidade e faria o título "Vendas 2023" mentir.

Decisão: **filtrar a janela de análise para 2023** no ETL (`Date.Year([Data]) = 2023`), removendo as 2 linhas → **998 transações**. A escolha foi registrada, e o filtro ficou no Power Query (reprodutível) para garantir que Excel e Power BI reconciliassem no mesmo número.

## Checagens de integridade

- **`Valor Total = Quantidade × Preço Unitário`** → 0 divergências (base consistente).
- **`ID Transações` duplicado** → 0 (chave íntegra).
- Sem nulos. Idade entre 18 e 64. Categorias equilibradas (Vestuário 351 / Eletrônicos 342 / Beleza 307). Gênero 510 F / 490 M.

## Um aprendizado técnico de Excel

Ao montar as contagens de distintos, algumas fórmulas com `ÚNICO()` retornavam **1** em vez do valor real. O motivo: `ÚNICO` (função de _dynamic array_) não digere bem uma referência vinda de `INDIRECT` — colapsa por _implicit intersection_. A solução robusta e à prova de versão foi o idioma clássico:

```excel
=SOMARPRODUTO(1/CONT.SE(intervalo; intervalo))
```

que conta distintos sem depender de _dynamic arrays_. Pequeno detalhe, mas exatamente o tipo de coisa que só aparece fazendo — e que vale documentar.
