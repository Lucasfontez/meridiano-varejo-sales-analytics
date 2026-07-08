# 05 · Páginas e análise

O dashboard tem quatro páginas, e elas contam **uma história que converge** — não são quatro coleções de gráficos soltos. Cada página responde um ângulo, e os ângulos se amarram num fim.

---

## Página 1 · Visão Executiva

O panorama em 10 segundos. Quatro KPIs (Receita Total, Nº Transações, Ticket Médio, Itens Vendidos), a receita mensal com a média móvel sobreposta, e a receita por categoria.

**O que revela:**
- Receita de **R$ 454.470** em 998 transações, ticket médio ~R$ 455.
- A receita mensal **oscila em banda** (~R$ 30–50 mil) em torno de uma média móvel quase plana → **sem tendência nem sazonalidade forte**.
- As três categorias faturam quase o mesmo (Eletrônicos R$ 156,9 mil, Vestuário R$ 155,6 mil, Beleza R$ 142 mil) → **não há categoria-âncora**.

Um cuidado de honestidade aqui: o gráfico de linha, se ordenado por valor, "sobe" e finge crescimento. Ordenado cronologicamente (via _Sort by Column_), mostra a verdade — receita estável. A média móvel entra justamente para separar tendência real de ruído.

---

## Página 2 · Tempo & Sazonalidade

Quando vendemos. Receita acumulada no ano, receita por dia da semana, e um heatmap mês × categoria.

**O que revela:**
- A **acumulada é quase perfeitamente linear** — confirma o faturamento constante. A ausência de curva é a leitura.
- **Sábado concentra receita** (R$ 79 mil, contra R$ 54–69 mil dos outros dias). É a alavanca operacional mais clara: fim de semana. A barra de sábado ganhou o destaque em laranja (o accent reservado ao insight).
- O heatmap mostra que **Eletrônicos é a categoria que mais oscila** entre os meses; Beleza e Vestuário são estáveis.

---

## Página 3 · Demografia & Segmentação

Quem compra e o quê. Um combo de receita + ticket por faixa etária, o split por gênero, as transações por faixa etária, e um heatmap faixa × categoria.

**O que revela:**
- **Gênero não segmenta** — a receita é ~51% F / 49% M, equilibrada. Nem toda dimensão demográfica importa, e é honesto mostrar isso.
- Por faixa etária, os mais velhos (36–50 e 51+) trazem **mais receita**. Mas o **ticket médio é parecido** entre faixas (a linha do combo é quase plana quando o eixo começa em zero).
- As transações por faixa fecham a lógica: os 50+ fazem **mais transações** (312 vs 169 dos jovens). Ou seja, **faturam mais porque transacionam mais, não porque gastam mais por compra.**
- O heatmap mostra uma **leve afinidade dos 50+ por Eletrônicos** (10–12% da receita vs ~6% dos jovens) — tendência, não dominância.

Um detalhe técnico que valeu a pena: o combo tinha eixo secundário começando em 450, o que fazia a linha do ticket "despencar" e mentir. Começando o eixo em **zero**, a linha fica quase plana — que é a realidade (o ticket varia pouco entre faixas).

---

## Página 4 · Valor do Cliente

Onde está o valor — e o achado central do projeto. KPIs da tese (Nº Clientes 998, Taxa de Recompra 0%, % Compra Única 100%), um histograma de distribuição de valor, a receita por faixa de valor, e o alto valor por quantidade.

### A tese de compra única
Os três KPIs cravam: 998 clientes, 0% de recompra, 100% de compra única. A base é 1:1 — e isso é o achado, não uma falha. Num negócio real, um repeat rate de 0% seria a prioridade estratégica número um.

### O achado central — por eliminação

**80% da receita vem de 30% das transações** (o tier "Alto", ≥ R$ 600). A pergunta: o que caracteriza essas transações de alto valor? Testei três hipóteses:

| Hipótese | Teste | Resultado |
|---|---|---|
| **Categoria** (produto caro) | Distribuição do tier Alto por categoria | Beleza 32% / Eletrônicos 34% / Vestuário 34% — **quase igual → refutada** |
| **Idade** (perfil premium) | Distribuição do tier Alto por faixa etária | Gradiente fraco (51+ lidera por pouco) → **refutada como driver forte** |
| **Quantidade** | Receita do tier Alto por qtd de itens | Qtd 1 **nunca** atinge alto valor; Qtd 4 sozinha = **45%** da receita do tier → **confirmada** |

**Conclusão:** o alto valor é dirigido por **volume de itens por transação** — não por categoria nem por perfil de cliente. A alavanca de receita é aumentar unidades por compra (cross-sell / bundle), não mix de produto ou segmentação demográfica.

O histograma sustenta visualmente: distribuição _right-skewed_, com uma massa de transações de tíquete baixo e uma cauda de tíquete alto puxada por quantidade.

---

## A narrativa que amarra tudo

Lendo as quatro páginas em sequência:

1. **Executiva:** receita estável, categorias empatadas.
2. **Tempo:** sem sazonalidade; o único pico é o sábado.
3. **Demografia:** o segmento 50+ é o motor de receita — via volume de transação, não ticket.
4. **Valor:** o alto valor vem de quantidade de itens, não de categoria nem perfil.

Tudo converge para uma leitura de negócio única: **o faturamento da Meridiano é estável e pulverizado; a alavanca de crescimento não é vender produto caro nem focar um segmento, é aumentar itens por transação — com o cliente 50+ (que já transaciona mais e tem afinidade por Eletrônicos) como público prioritário.**

Isso é o que separa um dashboard descritivo de uma análise: os números não só aparecem, eles **respondem a um "e daí?"**.

## Limitações (honestas)

- **Base sintética:** a uniformidade entre categorias e a fraca correlação demográfica indicam geração sem viés plantado. Num negócio real, esperaríamos categoria/demografia discriminando o alto valor — aqui, o valor foi de **método**.
- **1 ano de dados:** sem comparação YoY; a análise temporal se apoia em sazonalidade intra-ano, dia da semana e média móvel.
- **~1.000 transações:** variações mensais ruidosas, suavizadas pela média móvel.
- **Base 1:1:** RFM, cohort e CLV não se aplicam — a tese de compra única foi a leitura honesta.
