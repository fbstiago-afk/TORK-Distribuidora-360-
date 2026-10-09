# QA — TORK Distribuidora 360°

Testes executados em 08/10/2026 no Power BI Desktop (esquema de relatório 3.3.0, modelo em nível de compatibilidade 1606), com o projeto `TORK_360.pbip` aberto e os dados carregados. Contexto dos testes numéricos: Ano = 2026, corte em 08/10/2026.

## Resultado por critério do prompt (seção 12)

| # | Critério | Resultado | Evidência |
|---|---|---|---|
| 1 | CSVs carregados, sem FKs órfãs, sem duplicação | Aprovado | 12 tabelas com as contagens do `AUDITORIA_BASE.csv` (720.000 vendas, 720.000 títulos, 414.400 snapshots, 75.000 compras, 680 metas, 48.000 visitas). Chaves primárias sem duplicidade e zero órfãos nas FKs de dimensão. Datas fora do calendário só em colunas de papel secundário (previsão de entrega, vencimento, pagamento, próxima ação), que não usam relação ativa. |
| 2 | Medidas reconciliadas | Aprovado | `validacao_numerica.csv`: 43 medidas comparadas com cálculo independente em pandas. Diferença máxima de R$ 3,70 em R$ 10,6 milhões (efeito volume do PVM, por arredondamento de decimal fixo). Receita líquida total = títulos emitidos = R$ 512.499.209,19; por pedido a diferença máxima é zero. |
| 3 | Estoque não somado no tempo | Aprovado | Todas as medidas de saldo filtram `fEstoque[Data] = [Data Último Snapshot]`. Snapshot exibido no cabeçalho e no KPI (03/10/2026 para o corte de 08/10/2026). |
| 4 | Futuro separado do realizado | Aprovado | Medidas-base filtram a data do fato até `[Data Corte]`. R$ 11.101.437,18 em 15.324 vendas posteriores a 08/10/2026 ficam fora do realizado e só entram no modo demonstração. Projeção rotulada como simulação. |
| 5 | Interações | Aprovado com ressalvas | Testados na tela: navegação entre páginas, painel de filtros (abrir, selecionar Filial, aplicar e fechar com a seleção preservada), sincronização do segmentador entre páginas, alternância de métrica (parâmetro desconectado), alternância de dimensão (parâmetro de campo), alternância Cascata ↔ PVM (bookmarks), botão Limpar filtros e os quatro tooltips de página. Projeto fechado e reaberto do disco sem erros nem avisos. Não testados: drillthrough (não construído) e comportamento no Power BI Service. |
| 6 | HTML/SVG renderizado | Aprovado no Desktop | Visual HTML Content carregado do AppSource; todos os componentes renderizaram. Service e exportação para PDF/PowerPoint não testados. |
| 7 | Sem visual vazio ou texto cortado | Aprovado com ressalvas | Quatro páginas revisadas por captura de tela a 80% de zoom. Tabelas largas podem exibir barra de rolagem horizontal em telas menores. |
| 8 | Performance | Aprovado | Performance Analyzer com atualização de todos os visuais, visual mais lento por página: Visão Executiva 0,72 s (faixa de KPIs); Comercial & Clientes 0,43 s (carteira acionável); Rentabilidade & Ações 1,08 s (faixa de KPIs); Estoque & Supply 0,95 s (itens críticos, depois da correção). Gargalo encontrado e corrigido: a tabela de itens críticos levava 7,4 s porque o filtro de visual fazia o Power BI calcular todas as medidas nas 2.800 combinações SKU × filial, e a medida de recebimentos previstos gerava filtros de data por combinação. Com medidas condicionadas a um teste barato de criticidade e a medida de recebimentos reescrita, o visual caiu para 0,95 s no Performance Analyzer. |
| 9 | ABC/XYZ, churn, OTIF, PVM, forecast, prioridade | Aprovado | PVM reconcilia com a variação total (diferença de controle de R$ 0,10). Inativos, reativados, OTIF, projeção e itens em risco conferem com o controle. ABC × XYZ conferido visualmente (700 SKUs distribuídos nas nove células, soma das participações = 100%). |
| 10 | Identidade TORK | Aprovado | Fundos e logo fornecidos aplicados; nenhum texto de exibição usa o nome Voltz. Os SKUs mantêm o prefixo `VOL-` porque são dados originais. |

## Testes do adendo (seção J)

| # | Teste | Resultado |
|---|---|---|
| 1 | Imagens reais aplicadas | `Dashboard TORK_ Distribuição em Movimento.png` na página Início; `Fundo Automotivo Corporativo TORK.png` nas quatro páginas analíticas. Ambas convertidas para JPG 1600×900. |
| 2 | Landing e botão INICIAR ANÁLISE | Página criada com botão nativo transparente sobre a arte do botão e quatro atalhos sobre os ícones. Botão INICIAR ANÁLISE testado: leva à Visão Executiva. Os quatro atalhos usam a mesma ação de navegação e não foram clicados um a um. |
| 3 | Painel de filtros oculto em todas as páginas | Criado nas quatro páginas. Aberto e fechado na Visão Executiva e na Rentabilidade, com a seleção preservada. |
| 4 | Barra HTML de filtros ativos | Testada sem filtro, com filtro de ano e com filial única. Seleção múltipla e filtro cruzado não foram testados na tela. |
| 5 | Títulos centralizados e alinhamento de tabelas | Títulos centralizados por tema e por visual. Cabeçalhos de coluna centralizados; primeira coluna de texto à esquerda e números à direita, que é o comportamento nativo. |
| 6 | Matriz com minigráfico e barras | Matriz Filial › Vendedor com bullet de meta, barra de MC % e sparkline em SVG. Ordenação não testada. |
| 7 | Quatro tooltips | Vendedor/filial, Cliente, Produto e Fornecedor testados na tela, com o contexto da linha ou barra apontada. |
| 8 | Navegação, alternâncias e filtros preservados | Testado (ver critério 5). |
| 9 | Textos de desempenho por DAX | Todos os números, títulos de tendência, alertas e cards vêm de medidas. São estáticos os nomes de página, rótulos de botão e títulos descritivos de visuais. |
| 10 | HTML em tamanhos diferentes | Testado em três tamanhos de contêiner: faixa larga (1568 px), painel médio (548 a 760 px) e tooltip (380 px). Layout mobile não foi criado. |

## Pendências reais

- Drillthrough de Cliente, SKU, Filial e Fornecedor não foi construído; os tooltips de página cobrem parte da necessidade.
- Recursos de agosto/2026 (rosca com valor central, novo seletor de datas, cabeçalhos fixos de matriz, tema Fluent 2) não foram usados: os nomes das propriedades no PBIR não puderam ser confirmados sem documentação e não foram inventados.
- Linhas de referência e quadrantes nos gráficos de dispersão não foram configurados.
- Publicação no Service, RLS, exportação e layout mobile não fazem parte desta entrega.
