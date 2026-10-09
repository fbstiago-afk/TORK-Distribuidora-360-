# Matriz de compatibilidade — recursos usados

Ambiente verificado: Power BI Desktop que grava relatório no esquema 3.3.0 (maio/2026 ou posterior) e modelo em nível 1606.

| Recurso | Estado | Onde foi usado | Evidência | Fallback |
|---|---|---|---|---|
| PBIP + PBIR + TMDL | GA | Projeto inteiro (9 páginas, 131 visuais, 10 bookmarks, 23 tabelas, 220 medidas) | Abre e salva no Desktop | Salvar como PBIX |
| Aplicar alterações externas ao projeto | Disponível na versão instalada | Fluxo de edição dos arquivos | Usado em toda a construção | Fechar e reabrir |
| Tema JSON | GA | `TORK360.json` como tema personalizado | Cores e títulos centralizados aplicados | Formatação por visual |
| Visual HTML Content (AppSource) | Visual certificado de terceiro | Cabeçalho, chips, KPIs, alertas, heatmap, Pareto, cascata, PVM, Decision Center, tooltips | Renderizado no Desktop | Medidas numéricas em cartões e tabelas nativos |
| Medida como imagem SVG (categoria ImageUrl) | GA | Bullets, barras e sparklines em tabelas e matrizes nativas | Renderizado; exige `imageWidth` e cores em `rgb()` | Barras de dados nativas |
| Parâmetro de campo | GA | Dimensão do ranking (Filial, Categoria, Vendedor, Canal, Segmento) e indicador do ranking de vendedores | Alternância testada | Bookmarks |
| Cadeia de formato dinâmica | GA | Medidas `Hero …` (R$ ou unidades) | Eixo muda com a métrica | Medidas separadas |
| Funções WINDOW/TOPN/GROUPBY em medidas | GA | Pareto, ABC, PVM | Resultados conferidos | — |
| Segmentador nativo em blocos | GA | Métrica, dimensão, indicador, ordenação | Testado | Lista suspensa |
| Segmentador sincronizado | GA | Ano, período, filial, vendedor, categoria, cliente, modo | Filial sincronizou entre páginas | Filtro de relatório |
| Bookmarks com dados desativados | GA | Painel de filtros e Cascata ↔ PVM | Seleção de filtros preservada ao abrir e fechar | — |
| Tooltip de página de relatório | GA | Quatro páginas de tooltip | Os quatro testados na tela | Tooltip nativo |
| Formatação condicional por medida | GA | Cor das barras, cor do status, cor da variação de margem | Renderizado | Regras fixas |
| Rosca com valor central, novo seletor de datas, cabeçalhos fixos, Fluent 2 | Não verificado | Não usados | — | Visuais atuais |
| Cálculos visuais, UDFs DAX | Não usados | — | — | Medidas do modelo |
