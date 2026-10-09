# Biblioteca HTML/SVG — TORK 360°

Os componentes são gerados por medidas DAX e renderizados pelo visual **HTML Content** (AppSource), sem scripts, sem fontes remotas e sem chamadas externas. O CSS comum está em `tork_componentes.css` e na medida `HTML | CSS`.

| Componente pedido | Medida | Onde aparece |
|---|---|---|
| KPI_Executive | `HTML | KPIs Executiva`, `Comercial`, `Estoque`, `Rentabilidade` | Faixa de KPIs das quatro páginas |
| Mini_Trend | `SVG | Spark …` (HTML) e `IMG | Spark …` (imagem para tabela nativa) | KPIs, tooltips, tabelas e matrizes |
| Progress_Target | `IMG | Bullet Meta` e barra de progresso do KPI de meta | Matriz filial › vendedor, KPI de meta e de positivação |
| Insight_Alert | `HTML | Insights Executiva` | Visão Executiva |
| Status_Badge | selo de severidade dos alertas e selo de prioridade do Decision Center | Visão Executiva e Rentabilidade |
| Supplier_Score | `IMG | Barra Score`, `IMG | Barra OTIF` | Tabela de fornecedores |
| Stock_Risk | `IMG | Barra Cobertura`, `HTML | Heatmap ABC XYZ`, `HTML | Pareto Excesso` | Estoque & Supply |
| Decision_Card | `HTML | Decision Center` | Rentabilidade & Ações |
| Cascata e ponte | `HTML | Waterfall Margem`, `HTML | Ponte PVM` | Rentabilidade & Ações |
| Cabeçalho e chips | `HTML | Cabeçalho …`, `HTML | Filtros Ativos` | Todas as páginas analíticas |
| Tooltips | `HTML | Tooltip Cliente`, `Produto`, `Fornecedor`, `Vendedor` | Páginas de tooltip |

Regras seguidas:
- Todo número vem de medida no contexto de filtro do visual; não há valor fixo de desempenho.
- Textos vindos de dados passam por escape de `&`, `<` e `>`.
- Animações são só de entrada (`tkin`), preenchimento (`tkgrow`) e três pulsos no selo crítico (`tkpulse`), e são desligadas por `prefers-reduced-motion`.
- As imagens `IMG | …` usam `rgb()` no lugar de `#` porque o caractere `#` codificado como `%23` não foi interpretado pelo Power BI Desktop nos testes.
- Fallback nativo: as mesmas grandezas existem como medidas numéricas (pastas 01 a 08) e podem ser colocadas em cartões e tabelas nativas se o visual HTML Content não puder ser carregado.
