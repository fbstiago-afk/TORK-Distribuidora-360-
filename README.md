# TORK Distribuidora — Inteligência 360°

Painel executivo em Power BI que une vendas, clientes, estoque, fornecedores e margem de uma distribuidora automotiva com 4 filiais, 2.000 clientes e 700 SKUs. Projeto de portfólio em formato **PBIP** (relatório PBIR + modelo semântico TMDL), com tema JSON e componentes HTML/SVG gerados por DAX.

> Todos os dados são **sintéticos**. A base foi documentada como "Voltz" e é exibida como TORK. Nenhum cliente, fornecedor ou valor é real.

![Página inicial](assets/tork_inicio.jpg)

## Páginas

| Página | Pergunta | Conteúdo |
|---|---|---|
| Início | — | Apresentação institucional e acesso ao dashboard |
| Visão Executiva | O que mudou e onde agir? | KPIs com sparklines, tendência Realizado × Ano anterior × Meta × Projeção, alertas priorizados, ranking por dimensão, matriz filial › vendedor |
| Comercial & Clientes | Quem cresce, cai ou destrói margem? | Segmentos RFV, compra × potencial estimado, cross-sell, ranking de vendedores, carteira acionável |
| Estoque & Supply | Onde faltará ou sobrará produto? | Matriz ABC × XYZ, Pareto do excesso, itens críticos, fornecedores com OTIF e score |
| Rentabilidade & Ações | O que fazer agora? | Cascata da margem, ponte preço-volume-mix, rentabilidade por categoria, Decision Center |

Em todas as páginas analíticas há cabeçalho HTML, barra de filtros ativos, navegação por botões e painel de filtros oculto. Quatro páginas de tooltip (cliente, produto, fornecedor, vendedor) completam o relatório.

## Como abrir

1. Clone o repositório.
2. Descompacte as três tabelas grandes, que estão em `.gz` por causa do limite de 100 MB por arquivo do GitHub:
   ```bash
   gunzip -k dados/fVendas.csv.gz dados/fFinanceiro.csv.gz dados/fEstoque.csv.gz
   ```
   No Windows, o 7-Zip faz o mesmo. Os arquivos `.csv` devem ficar na pasta `dados`.
3. Abra `TORK_360.pbip` no Power BI Desktop (versão com suporte a PBIR).
4. Em **Transformar dados**, ajuste o parâmetro `PastaDados` para o caminho da pasta `dados` no seu computador, terminando com `\`.
5. Atualize os dados. O visual **HTML Content** é baixado do AppSource na abertura.

## Estrutura

| Pasta | Conteúdo |
|---|---|
| `TORK_360.SemanticModel` | Modelo TMDL: 23 tabelas, 25 relacionamentos, 220 medidas |
| `TORK_360.Report` | Relatório PBIR: 9 páginas, 131 visuais, bookmarks, tema e imagens |
| `dados` | 12 tabelas em CSV (três compactadas) e a documentação da base |
| `dax` | Todas as medidas com a regra de negócio |
| `tema` | Tema JSON TORK |
| `html` | CSS e índice da biblioteca de componentes HTML/SVG |
| `assets` | Fundos usados no relatório |
| `documentacao` | Decisões de modelagem e matriz de compatibilidade |
| `qa` | Checklist de testes e validação numérica |

## O caso

**A distribuidora que vendia quase o mesmo e ganhava 43% menos.** Números do acumulado de 2026 até 08/10, na base sintética:

- A margem de contribuição caiu de 19,0% para 11,5%, o equivalente a R$ 10,0 milhões, enquanto o faturamento recuou só 6,6%.
- Baterias, 57% da receita, foi de 18,3% para 6,6% de margem: o custo unitário subiu 15% e o desconto médio passou de 7,4% para 10,3%.
- O estoque soma R$ 555 milhões, 1.323 dias de cobertura, com 95% acima de 120 dias. Ao mesmo tempo, 168 combinações de produto e filial estão zeradas.
- Só 53,6% dos pedidos de compra chegam no prazo e completos.
- R$ 18,7 milhões estão vencidos, 59% do saldo em aberto.

## Validação

43 medidas foram comparadas com um cálculo independente sobre os CSVs (`qa/validacao_numerica.csv`). Interações, tooltips e desempenho estão registrados em `qa/CHECKLIST_QA.md`, com as pendências conhecidas.

## Limitações

- A base não tem despesas fixas: o painel vai até a margem de contribuição e não apresenta EBITDA.
- Potencial de cliente, share of wallet e gap são estimativas cadastrais, não mercado medido.
- Drillthrough, layout mobile e publicação no Power BI Service não fazem parte desta versão.
