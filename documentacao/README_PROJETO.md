# TORK Distribuidora — Inteligência 360°

Projeto Power BI em formato PBIP (relatório PBIR + modelo TMDL) sobre a base sintética de distribuição automotiva (documentada como Voltz, exibida como TORK). Todos os dados são fictícios.

## Como abrir

1. Abra `TORK_360.pbip` no Power BI Desktop.
2. Os CSVs são lidos de `E:\Projetos BI\09 - Distribuidora 360\dados\`. Se mover o projeto, altere o parâmetro `PastaDados` em Transformar dados.
3. O visual HTML Content é carregado do AppSource na abertura; é preciso estar conectado.

## Estrutura de pastas

| Pasta | Conteúdo |
|---|---|
| `TORK_360.SemanticModel` | Modelo TMDL: 23 tabelas, 25 relacionamentos, 220 medidas |
| `TORK_360.Report` | Relatório PBIR: 5 páginas visíveis, 4 páginas de tooltip, 10 bookmarks, tema e imagens |
| `dados` | 12 CSVs, `README_BASE_DADOS.md`, `MODELO_RELACIONAL.md`, `AUDITORIA_BASE.csv` |
| `dax` | `MEDIDAS_TORK_360.dax`: todas as medidas com a regra de negócio |
| `tema` | `TORK360.json` |
| `html` | CSS dos componentes e índice da biblioteca HTML/SVG |
| `assets` | Fundos usados no relatório |
| `documentacao` | Este arquivo e a matriz de compatibilidade |
| `qa` | Checklist de testes e validação numérica |

## Páginas

| Página | Pergunta | Blocos |
|---|---|---|
| Início | — | Arte institucional, botão INICIAR ANÁLISE, quatro atalhos, linha dinâmica com data de corte e tamanho da base |
| Visão Executiva | O que mudou e onde agir? | 7 KPIs; tendência Realizado × Ano anterior × Meta × Projeção com máximo, mínimo e quedas acima do limiar; alertas priorizados; ranking por dimensão; dispersão crescimento × margem; matriz filial › vendedor |
| Comercial & Clientes | Quem cresce, cai ou destrói margem? | 8 KPIs; carteira por segmento RFV; compra × potencial estimado; cross-sell por categoria; ranking de vendedores com indicador alternável; carteira acionável |
| Estoque & Supply | Onde faltará ou sobrará produto? | 8 KPIs; matriz ABC × XYZ; Pareto do excesso por categoria; itens críticos SKU × filial; fornecedores com OTIF e score; atraso mensal |
| Rentabilidade & Ações | O que fazer agora? | 7 KPIs; cascata da margem ↔ ponte PVM; clientes faturamento × margem; rentabilidade por categoria; Decision Center |

Em todas as páginas analíticas: cabeçalho HTML com período efetivo e microindicador, barra de filtros ativos, navegação por botões nativos e painel de filtros oculto aberto pelo botão **Filtros**.

## Decisões de modelagem

- **Data de corte.** `[Data Corte]` é 08/10/2026. As medidas-base filtram a data do próprio fato até o corte, então nada posterior entra como realizado. O segmentador *Modo de análise* troca para o horizonte sintético completo (31/10/2026) e a barra de filtros avisa.
- **Ano anterior comparável.** As medidas `… LY` deslocam apenas as datas até o corte: outubro de 2026 (8 dias) é comparado com 1 a 8 de outubro de 2025.
- **Metas.** `fMetas` liga-se a `dMes`, não ao calendário. `[Meta Faturamento]` proporcionaliza a meta do mês pelos dias úteis decorridos; `[Meta Faturamento Mês Cheio]` é usada no gráfico mensal. Metas não respondem a filtros de cliente ou produto.
- **Projeção.** Realizado no mês do corte + receita por dia útil dos últimos 60 dias × dias úteis restantes. É simulação e está rotulada assim.
- **Estoque.** Posição sempre no último snapshot semanal até o fim do período. Cobertura = valor em estoque ÷ CMV diário de 90 dias. Excesso = valor acima de 120 dias de cobertura por SKU × filial.
- **Financeiro.** `SaldoAberto` do arquivo está apurado em 31/10/2026. As medidas recalculam a posição no corte: título pago depois do corte conta pelo valor integral; título sem pagamento conta pelo saldo.
- **Supply.** OTIF sobre pedidos entregues até o corte. Cada SKU tem um único fornecedor na base, então o componente de preço do Supplier Score usa o score cadastral, não preço relativo.
- **Clientes.** Carteira filtrada por vendedor e filial via `TREATAS`, porque as dimensões não se relacionam entre si. RFV é coluna calculada na data de corte padrão. Potencial, share e gap são estimativas cadastrais.
- **ABC × XYZ.** ABC pela receita do período sobre todo o portfólio (80% / 95%). XYZ pelo coeficiente de variação da quantidade mensal (0,30 / 0,60), com mínimo de 6 meses.
- **PVM.** Preço, volume e mix sobre SKUs vendidos nos dois períodos; novos e descontinuados separados; reconcilia com a variação total.
- **Sem EBITDA.** A base não tem despesas fixas. O relatório para na margem de contribuição operacional.

## Limitações conhecidas

- Cada pedido da base tem um único item, então ticket por pedido equivale a valor por item.
- O atraso médio dos meses mais recentes só é exibido quando 95% dos pedidos já foram entregues, para evitar o viés das entregas rápidas.
- A base tem estoque equivalente a cerca de 3,6 anos de venda; por isso quase todo o saldo é classificado como excesso pela regra de 120 dias.
- Pendências de construção estão em `qa/CHECKLIST_QA.md`.
