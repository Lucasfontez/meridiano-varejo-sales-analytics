# 01 · Contexto e escopo

## Como o projeto começou

Antes de escolher, avaliei três datasets de vendas do Kaggle — ZARA, Amazon e Retail Sales. Descartei os dois primeiros por um motivo técnico, não de gosto: o de ZARA é um _snapshot_ raspado num único ponto no tempo (sem série temporal) e o "Amazon Sales" não tem venda nem data — é um catálogo de preço e reputação. Só o **Retail Sales** é transacional de verdade, com **data + demografia**, que é onde o Power BI mostra o que tem de melhor (_time intelligence_, segmentação, valor do cliente).

Escolher pelo que o dado **realmente permite**, e não pelo nome do arquivo, já foi a primeira decisão de analista do projeto.

## O cliente (fictício)

Para dar contexto de negócio ao trabalho, criei um cliente fictício: a **Meridiano Varejo**, uma rede de médio porte que vende três categorias — Eletrônicos, Vestuário e Beleza. O cenário: a área comercial tem um ano de dados de vendas parado numa planilha e decide no _feeling_ — quando fazer promoção, para quem direcionar campanha, em qual categoria investir. Ninguém consegue olhar para o dado e responder pergunta simples.

Trabalhar a partir de um _brief_ (mesmo que simulado) força a pensar no "para quê", não só no "como".

## As perguntas de negócio

Tudo no projeto existe para responder cinco perguntas:

1. Quanto vendemos e como a receita evolui ao longo de 2023?
2. Quais categorias puxam a receita e como o mix se comporta no tempo?
3. Quem são os clientes (idade/gênero) e isso muda o quanto/o que compram?
4. Temos clientes que voltam a comprar, ou é tudo compra única? Onde a receita se concentra?
5. Quando são os picos (mês, dia da semana)? Há sazonalidade a aproveitar?

## Escopo — decisões conscientes

O projeto foi mantido em **nível júnior por posicionamento de portfólio**, não por limitação. Ficaram deliberadamente de fora: segurança por perfil (RLS), atualização incremental, modelos preditivos e _composite models_. São padrões de nível pleno/sênior que não agregam ao objetivo (fundamentos bem executados) e consumiriam tempo que foi melhor investido em modelagem limpa e narrativa.

A régua foi: **fundamento bem feito comunica mais do que técnica avançada mal amarrada.**

## Ferramentas

- **Excel** (Power Query, Power Pivot, DAX) e **Power BI** como co-principais.
- Excel teve ênfase de prática: como Power Query e Power Pivot compartilham as engines do Power BI, cada hora de Excel transferiu direto para o BI. O modelo foi ensaiado no Excel (Power Pivot) antes de migrar.
- **SQL** ficou como trilha opcional, para comparar abordagens de validação.

## Método de trabalho

O projeto foi conduzido de forma iterativa, etapa a etapa, com um roteiro de nove estágios definido antes da execução: entendimento do negócio → profiling → ETL → modelagem → medidas → visualização → narrativa → publicação → empacotamento. Cada etapa só avançava com a anterior validada.
