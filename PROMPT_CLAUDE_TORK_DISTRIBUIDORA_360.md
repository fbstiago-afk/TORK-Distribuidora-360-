# PROMPT MESTRE — TORK DISTRIBUIDORA | POWER BI 360°

## PAPEL E MISSÃO
Atue como arquiteto sênior de Power BI, especialista em modelagem tabular, DAX, Power Query, design de dashboards, operações B2B, supply chain e UX analítica. **Implemente de fato** um projeto de portfólio premium chamado **TORK Distribuidora — Inteligência 360°**, usando os CSVs e a documentação disponíveis na pasta do projeto. Não entregue somente recomendações ou mockups: produza artefatos funcionais, medidas, tema, páginas, interações e documentação, na medida em que o ambiente e as ferramentas conectadas permitirem.

**Princípio:** sofisticação visual sem perder rigor: cada insight deve ter métrica verificável, contexto, ação e impacto; cada efeito visual deve preservar desempenho, acessibilidade e compatibilidade com o Power BI Desktop e Service.

## 0. INSUMOS E LEITURA OBRIGATÓRIA
1. Inspecione toda a pasta do projeto, os arquivos CSV, `README_BASE_DADOS.md`, `MODELO_RELACIONAL.md`, `VALIDACAO.json` e os anexos/referências de visuais modernos do Power BI. Leia as skills/instruções locais de Power BI, HTML Content, DAX, PBIP, TMDL e design, **se realmente existirem**; não afirme que foram lidas sem acesso.
2. A base está nomeada **Voltz** na documentação original, mas a marca final do dashboard é **TORK DISTRIBUIDORA**. Mantenha os nomes das tabelas e colunas dos CSVs; substitua a identidade textual de exibição por TORK, sem alterar valores, chaves ou dados sem necessidade.
3. Use o logo e o mockup TORK fornecidos como referência visual: **azul-marinho #0B1F33, vermelho #E31E24, cinza #6C737D, off-white #F5F6F7**, com Montserrat (ou fallback disponível). Marca automobilística corporativa, precisão, velocidade e confiabilidade. NÃO reproduza dados pessoais, contatos ou domínios fictícios vistos em mockups como se fossem reais.
4. A base contém 12 tabelas: `dCalendario`, `dCliente`, `dFilial`, `dFornecedor`, `dProduto`, `dVendedor`, `fCompras`, `fEstoque`, `fFinanceiro`, `fMetas`, `fVendas`, `fVisitas`. Período 01/01/2024–31/10/2026. Volume aproximado: 720 mil itens de vendas, 720 mil títulos financeiros, 414,4 mil snapshots semanais de estoque, 75 mil compras e 48 mil interações comerciais.
5. **Data de referência: 08/10/2026.** Há dados sintéticos até 31/10/2026, inclusive no futuro. Defina claramente um parâmetro `DataCorteAnalise` (por padrão 08/10/2026) e não apresente dados posteriores como realizados. Permita modo demonstração com horizonte sintético completo, rotulado explicitamente.
6. Se algum anexo visual mencionado não estiver disponível, registre a ausência e siga as especificações abaixo; não invente seu conteúdo.

## 1. ENTREGA TÉCNICA ESPERADA
- Projeto Power BI editável preferencialmente **PBIP/PBIR + modelo semântico TMDL**, se o ambiente suportar; alternativamente PBIX editável. Não alegue ter gerado PBIX se só produziu arquivos de texto.
- Pasta organizada: `/dados`, `/modelo`, `/dax`, `/tema`, `/html`, `/assets`, `/documentacao`, `/qa`.
- Tema JSON global; medidas DAX organizadas em display folders; código HTML/SVG e CSS compatível com o visual de destino; documentação das interações, bookmarks, parâmetros e filtros; roteiro de testes e capturas das quatro páginas.
- Se não puder manipular o Desktop ou gravar PBIP diretamente, entregue **arquivos prontos para implementação** (DAX, M, JSON, HTML, roteiro operacional) e separe claramente concluído x pendente. Não pare na proposta.
- Criar quatro páginas analíticas e, opcionalmente, uma landing page de navegação que **não conta** como quinta página analítica.

## 2. AUDITORIA DOS DADOS ANTES DO DESIGN
- Validar nomes, tipos, granularidade, PK/FK, cardinalidade, datas, nulos, unicidade, integridade e consistência monetária.
- Conferir `SUM(fVendas[ReceitaLiquida])` versus `SUM(fFinanceiro[ValorTitulo])` **no conjunto completo** e por `PedidoID` quando aplicável; não comparar indevidamente os mesmos totais sob datas diferentes sem explicar o contexto.
- `fEstoque` é **snapshot SEMANAL aos sábados**: jamais somar saldos de várias semanas para obter estoque atual. Utilizar último snapshot disponível no contexto, e alertar quando o dado estiver defasado em relação à data de corte.
- `fEstoque[Ajuste]` pode representar suprimento emergencial **sem compra correspondente**: não atribuir toda entrada/ajuste a `fCompras`.
- `fVisitas` não tem ligação causal comprovada com pedidos; conversão por janela temporal é aproximação, rotulada.
- `dCliente[PotencialMensalEstimado]` é uma hipótese sintética, não market share medido; share of wallet e gap são **estimados**.
- A base **não possui DRE contábil completa nem despesas fixas**: não apresentar EBITDA real ou lucro líquido auditado. Usar margem de contribuição operacional observável, com custos disponíveis.
- `dProduto[CurvaABCBase]` é ilustrativa; calcular ABC dinâmica a partir das vendas no período selecionado. XYZ a partir da variabilidade temporal, com tratamento para demanda zero e amostras insuficientes.
- Metas `fMetas` estão no grão mês/vendedor; não criar relacionamento direto com `dCalendario[AnoMes]` não único. Construir `dMes` com uma linha por mês e chaves consistentes.

## 3. MODELO SEMÂNTICO E PERFORMANCE
- Modelagem estrela, dimensões filtrando fatos em direção única, sem relações bidirecionais indiscriminadas. Evitar caminhos ambíguos `dFilial→dVendedor→fVendas` e `dVendedor→dCliente→fVendas`; preservar filiais e vendedores diretamente nas fatos.
- `dCalendario[Data]` como calendário principal; datas alternativas de entrega, vencimento, pagamento e pedido via relações inativas + `USERELATIONSHIP` ou calendários de papéis, com justificativa. Dimensão `dPedido` apenas se necessária para drillthrough e sempre única.
- Desabilitar Auto Date/Time, definir tipos e ordenações, ocultar chaves técnicas, format strings, pastas de medidas, descrições, nomes amigáveis e convenções consistentes.
- Evitar colunas calculadas de alta cardinalidade e iteradores desnecessários; considerar agregações e medidas eficientes. Medir com Performance Analyzer/DAX Studio, se disponíveis.
- Preferir medidas-base reutilizáveis e cálculo de variações por `DIVIDE`; evitar medidas duplicadas. Definir políticas explícitas de denominador, BLANK, dias úteis e comparação de períodos parciais.
- Separar vendas realizadas até a data de corte de forecast. Para LY, comparar mesmo intervalo acumulado e não mês fechado versus parcial.
- Implementar e testar parâmetros de campo, bookmarks e botões antes de construir HTML complexo.

## 4. DESIGN SYSTEM E VISUAIS MODERNOS
**Canvas:** 16:9, sugerido 1600×900 (ou 1920×1080 se a legibilidade permitir). Fundo off-white nas páginas de análise, header azul-marinho, acento vermelho com uso comedido; cards claros, cantos de 10–14 px, bordas discretas, sombras suaves, espaçamento consistente (grid de 8 px). Visual premium e limpo, não futurismo excessivo.

**Tokens globais:** cores, tipografia, tamanhos, estados de seleção, bordas, raio, padding, títulos, labels, tooltips, cores semânticas (verde sucesso, âmbar alerta, vermelho risco, azul informativo). Priorizar tema JSON e configurações centralizadas/modern visual defaults suportadas pela versão instalada; só aplicar exceções locais quando necessárias. Não presumir que todos os novos recursos de preview estejam disponíveis.

**Navegação e interatividade:**
- Menu lateral ou superior compacto com logo TORK e quatro destinos; estado ativo, botões voltar, limpar filtros e ajuda contextual.
- Painel de filtros recolhível por botão/bookmarks, com Data, Filial, Vendedor, Categoria, Cliente, Fornecedor (exibir só onde fizer sentido). Botão de reset e indicador de filtros ativos.
- **Alternância real de visuais** por *Field Parameters*, *Bookmarks + Selection Pane*, botões e/ou *Visual Calculations* quando estáveis: ex. Faturamento ↔ Margem ↔ Volume; Mensal ↔ Semanal; Filial ↔ Categoria; R$ ↔ %; Top 10 ↔ Bottom 10; Tabela ↔ Gráfico. Preservar contexto e destacar opção selecionada.
- Drillthrough para Cliente, SKU, Filial e Fornecedor, sem criar novas páginas principais (usar páginas ocultas de detalhe se necessário).
- Tooltips customizados com tendência, contribuição, benchmark, comparação LY, risco e próxima ação. Configurar interações entre visuais para evitar filtros contraditórios.
- Explorar *small multiples*, *sparklines*, *decomposition tree*, *key influencers*, *reference lines*, *dynamic format strings*, *conditional formatting*, *calculation groups* (se apropriado), *visual calculations* e *on-object interaction* conforme disponibilidade e utilidade. Não adicionar recursos apenas por novidade.
- Navegação por teclado, contraste adequado, textos alternativos e opção de reduzir animações.

## 5. HTML / SVG / CSS: REGRAS DE COMPATIBILIDADE
Construir componentes de alto impacto com HTML/SVG **somente em visual que efetivamente renderize esse conteúdo** (por exemplo, visual HTML Content adequado e autorizado). Medidas DAX que retornam HTML **não** são interpretadas como HTML em cartões nativos do Power BI. Testar o visual no Desktop e Service, inclusive exportação e interações.

- CSS encapsulado por componente, sem dependência de scripts externos; não pressupor suporte a JavaScript, `<script>`, DOM dinâmico, eventos JS, fontes remotas, chamadas HTTP ou animações CSS completas no visual escolhido.
- SVG inline para indicadores, progress bars, bullet charts, micro-sparklines, heatmaps, mini waterfals e status. Preferir CSS simples e SVG estático com dados dinâmicos; animações discretas apenas após teste de suporte.
- Criar animações opcionais: barra de progresso, pulso suave em risco crítico, entrada/fade, preenchimento gradual de KPI. Garantir versão estática quando animação não renderizar ou prejudicar acessibilidade.
- Evitar HTML gigante em uma única medida; modularizar padrões, escape de textos, formatação `pt-BR` (R$, %, datas), sanitização e tratamento de BLANK/zero. Nunca embutir dados não filtrados em HTML.
- **Interação de alternar visuais deve ser implementada por recursos nativos do Power BI**; não simular clique HTML que não modifica o estado do relatório.
- Entregar biblioteca reutilizável: `KPI_Executive`, `Insight_Alert`, `Progress_Target`, `Status_Badge`, `Supplier_Score`, `Decision_Card`, `Mini_Trend`, `Stock_Risk` com variantes e fallback nativo.

## 6. PÁGINA 1 — VISÃO EXECUTIVA | “O que mudou e onde agir?”
**Header:** título, período, última atualização, filial, botão filtros, navegação.
**KPIs (distribuir em 2 linhas equilibradas ou uma linha com KPIs essenciais e alternância):** faturamento líquido, margem bruta R$, margem %, margem de contribuição, clientes ativos, ticket médio, estoque último snapshot, cobertura, ruptura estimada, inadimplência vencida, atingimento de meta, projeção de fechamento. Não empilhar 12 cards minúsculos: 6–8 em destaque e demais em faixa secundária alternável.

**Visual hero:** evolução Atual x LY comparável x Meta x Forecast com opção alternar faturamento/margem/volume e granularidade mês/semana. Linha de forecast visualmente diferenciada e método explicado.

**Complementares:** barras horizontais por filial/região; dispersão crescimento YoY x margem % com bolha de faturamento, linhas de referência e quadrantes; contribuição por categoria; ranking de desvios.

**Painel de insights automáticos:** 3–5 alertas dinâmicos, classificados por severidade e impacto (ex.: gap meta, compressão de margem, concentração, risco estoque, oportunidade comercial). Cada alerta deve ter cálculo e navegação para a página pertinente; sem números escritos manualmente.

## 7. PÁGINA 2 — COMERCIAL & CLIENTES | “Quem cresce, cai ou destrói margem?”
**KPIs:** faturamento, margem contribuição, ativos, novos, reativados, perdidos/inativos, positivação, frequência, mix médio, ticket e meta.

**Visuais principais:**
1. Funil comercial conceitualmente correto (carteira cadastrada → comprou no período → recorrente → alto valor), **não** chamar de funil de conversão CRM se não há oportunidades rastreadas.
2. Segmentação RFV/RFQ com regras documentadas, janelas temporais e grupos Campeões, Fiéis, Potenciais, Em risco, Hibernando e Perdidos.
3. Matriz Compra atual x Potencial mensal estimado com quadrantes e *share of wallet estimado* (`CompraMensalNormalizada / PotencialMensalEstimado`, limitar interpretação de >100%). Exibir gap estimado, nunca tratá-lo como demanda confirmada.
4. Cross-sell de categorias: compradores de Baterias sem Lubrificantes, etc.; quantidade de clientes, receita histórica e potencial hipotético claramente identificado.
5. Ranking vendedor alternável Faturamento ↔ Margem ↔ Positivação ↔ Mix ↔ Gap de Meta; gráfico de dispersão Venda x Potencial x Margem e análise de concentração.
6. Tabela de carteira acionável com cliente, último pedido, tendência, margem, frequência, mix, gap e prioridade, com drillthrough.

**Interações:** alternar RFV ↔ Churn ↔ Cross-sell; Top ↔ Bottom; período de inatividade parametrizável 30/60/90 dias; tooltip do cliente com mini-histórico e oportunidades.

## 8. PÁGINA 3 — ESTOQUE & SUPPLY | “Onde faltará ou sobrará produto?”
**KPIs:** valor de estoque no último snapshot, giro, cobertura, ruptura estimada, excesso, obsolescência, pedidos de compra em aberto, OTIF, lead time, atraso médio.

**Visuais principais:**
1. Matriz **ABC × XYZ dinâmica**, com receita/volume do período e coeficiente de variação de demanda; regras explícitas para amostra pequena e demanda zero. Heatmap com drillthrough SKU.
2. Tabela crítica SKU × filial: saldo último snapshot, venda média diária, cobertura em dias, lead time, estoque de segurança, ponto de reposição, risco e valor de capital parado. Distinguir ruptura observada em snapshot de risco projetado.
3. **Projeção de estoque** por SKU/filial: último saldo conhecido + recebimentos confirmados previstos − demanda projetada. Datas previstas de compra pendente não são entregas garantidas. Incluir intervalo de incerteza/cenário, data provável de ruptura, e sinalização de que é simulação.
4. Fornecedores: OTIF calculado sobre pedidos elegíveis, preço relativo por SKU comparável, qualidade como score cadastral, atraso, tendência de lead time. Supplier Score configurável: 40% OTIF, 25% preço, 20% qualidade, 15% lead time — **normalizar cada componente e mostrar fórmula**, sem somar unidades diferentes.
5. Pareto de capital parado, comparação de filiais e histórico de cobertura.

**Interações:** ABC ↔ XYZ ↔ ABC/XYZ; Ruptura ↔ Excesso; Valor ↔ Quantidade; fornecedor ↔ SKU; seleção de cenário de demanda (base/otimista/pessimista).

## 9. PÁGINA 4 — RENTABILIDADE & DECISION CENTER | “O que fazer agora?”
**KPIs:** receita líquida, CMV, margem bruta, margem de contribuição, frete rateado, comissões, desconto médio ponderado, rentabilidade por cliente e SKU. **Não exibir EBITDA real sem dados de despesas operacionais.**

**Visuais principais:**
1. Waterfall de economia da venda: Receita de tabela (bruta) − Descontos = Receita líquida − CMV − Frete − Comissão = Margem de contribuição. Não deduzir impostos inexistentes na base nem chamar de DRE contábil.
2. Scatter cliente: X faturamento, Y margem de contribuição %, bolha contribuição absoluta; destacar clientes de alta receita e baixa rentabilidade. Permitir alternar cliente ↔ SKU ↔ vendedor.
3. **Price–Volume–Mix** com metodologia documentada, conjunto comparável de SKUs, novos/descontinuados separados, reconciliação exata com a variação total e distinção de efeito preço vs volume vs mix; validar totais.
4. Análise de descontos x margem, rentabilidade por categoria, ranking dos maiores destruidores/criadores de contribuição.
5. **Decision Center**: cards de risco de ruptura, queda de margem, cross-sell e potencial comercial, com prioridade baseada em impacto econômico, confiança e urgência; mostrar `Evidência → Impacto estimado → Ação sugerida → Responsável sugerido → Drillthrough`. Não inventar impactos sem cálculo. Se estimativa indisponível, escrever “não estimado”.

**Interações:** waterfall ↔ PVM ↔ margem por categoria; ranking oportunidade ↔ risco; ordenar por impacto ↔ urgência ↔ confiança; drillthrough para evidência.

## 10. BIBLIOTECA MÍNIMA DE MEDIDAS DAX
Produzir medidas reais, testadas e organizadas em pastas:
- **Vendas:** Receita Bruta, Descontos, Receita Líquida, Quantidade, Pedidos Distintos, Ticket por Pedido, Preço Médio Ponderado, Receita LY Comparável, YoY %, MoM %, YTD, Meta, Gap Meta, Atingimento %, Forecast e intervalo de confiança/sinalização.
- **Margem:** CMV, Margem Bruta, Margem Bruta %, Frete, Comissão, Margem de Contribuição, MC %, Desconto % ponderado, Variação de Margem em p.p.
- **Clientes:** Ativos no período, Novos, Reativados, Perdidos (regra temporal), Frequência, Recência, Mix, RFV/RFQ, Share Estimado, Gap Estimado, Cross-sell.
- **Estoque:** Data Último Snapshot, Saldo Atual, Valor Atual, Venda Média Diária, Cobertura Dias, Giro, Estoque Parado, Ruptura Observada, Risco de Ruptura, Estoque Projetado.
- **Supply:** Compras Recebidas, Compras Pendentes, OTIF, On Time %, In Full %, Lead Time Real, Atraso Médio, Supplier Score.
- **Financeiro:** Títulos Emitidos, Valor Pago, Saldo em Aberto, Vencido na Data de Corte, Aging, DSO com definição explícita.
- **Decisão:** Impacto de Margem, Oportunidade Estimada, Prioridade, Severidade, Narrativa Dinâmica, PVM.

Incluir código DAX com tratamento de contexto, tabelas de suporte e comentários de regra de negócio. Não apenas listar nomes de medidas.

## 11. INSIGHTS DINÂMICOS E STORYTELLING
Para cada insight:
- Descrever a condição DAX de disparo, período, denominador, limiar e prioridade.
- Quantificar variação em R$, %, p.p. e comparação válida quando aplicável.
- Mostrar origem da evidência e confiança: observado, estimado ou simulado.
- Evitar mensagens genéricas e afirmações causais sem evidência.
- Usar cores semânticas e ícones discretos; a cor sozinha não deve transmitir status.

## 12. QUALIDADE, TESTES E CRITÉRIOS DE ACEITE
Executar checklist e documentar resultado:
1. CSVs carregados, sem FKs órfãs e sem duplicação indevida por relacionamento.
2. Medidas financeiras reconciliadas em total, mês, filial, vendedor, cliente e produto.
3. Saldos de estoque não somados ao longo do tempo; datas de snapshot visíveis.
4. Datas futuras não misturadas a realizado; forecast rotulado.
5. Interações, bookmarks, field parameters, slicers, reset, tooltips e drillthrough testados.
6. HTML/SVG renderizado no Desktop/Service conforme capacidade do visual; versão fallback onde necessário.
7. Nenhuma página com visual vazio por erro, texto cortado, contraste ruim ou excesso de informação.
8. Performance de cada página medida; identificar gargalos e melhorias.
9. Medidas de ABC/XYZ, churn, OTIF, PVM, forecast e prioridade verificadas com casos de teste.
10. Identidade TORK aplicada em todos os títulos, logos e materiais finais; dados originais preservados.

## 13. ORDEM DE EXECUÇÃO — TRABALHE POR ETAPAS
**Etapa A — Auditoria:** liste arquivos encontrados, esquema real, problemas, limitações e plano de correção. Em seguida execute correções justificadas.
**Etapa B — Modelo:** construa relações, calendário, dMes, medidas-base e QA numérico.
**Etapa C — Design:** aplique tema, tokens, grid, navegação, painel de filtros e biblioteca HTML/SVG.
**Etapa D — Página 1:** implemente, teste e capture a Visão Executiva.
**Etapa E — Página 2:** implemente Comercial & Clientes.
**Etapa F — Página 3:** implemente Estoque & Supply.
**Etapa G — Página 4:** implemente Rentabilidade & Decision Center.
**Etapa H — Entrega:** otimize, documente e apresente evidências, arquivos produzidos e pendências reais.

Em cada etapa entregue: arquivos alterados, medidas criadas, decisões técnicas, testes executados, limitações e próximo passo. **Não afirme que um visual ou recurso foi implementado sem ter sido criado e testado.** Se estiver conectado ao Power BI, implemente diretamente. Caso contrário, gere artefatos importáveis e instruções objetivas para concluir no Desktop.


## 14. ATUALIZAÇÕES DOS VISUAIS NATIVOS — REFERÊNCIA AGOSTO/2026

**Referência visual adicional:** imagem enviada pelo usuário, intitulada “Power BI — O que mudou nos visuais nativos? 6 novidades para explorar nos seus relatórios”, atribuída na própria arte ao Microsoft Learn / atualização de agosto de 2026. A imagem é uma representação ilustrativa: **confira no Power BI Desktop instalado e na documentação oficial quais opções existem na versão utilizada**, se estão em preview e quais exigem habilitação. Não invente propriedades, APIs, disponibilidade geral ou equivalência entre Desktop e Service.

**Implemente e teste, onde houver suporte, os seis aprimoramentos abaixo:**

1. **Segmentadores modernizados:** cantos arredondados, bordas, ícones e estados de seleção customizados. Criar slicers consistentes para Filial, Região, Categoria, Vendedor, Cliente e Período. Usar tokens globais TORK para raio, borda, preenchimento, tipografia, foco e seleção. Priorizar slicers nativos, com painel lateral recolhível e botão de limpar filtros.
2. **Seletor de datas:** explorar calendário, seleção de data única e filtros relativos, quando disponíveis. Oferecer seleção intuitiva para intervalo de análise e data de corte, distinguindo período de transações e data de snapshot de estoque. Evitar sincronização enganosa de slicers com fatos de diferentes granularidades.
3. **Gráfico de rosca com valor central:** usar opção nativa de valor central dinâmico quando suportada e configurável. Aplicações: participação de categorias, composição de receita, carteira ativa/inativa e status de estoque. O valor central deve acompanhar os filtros e ter significado claro (total ou percentual, devidamente rotulado). Se a versão não oferecer o recurso, usar alternativa compatível sem simular uma funcionalidade nativa inexistente.
4. **Matrizes aprimoradas:** testar expansão/recolhimento hierárquico de colunas e fixação de cabeçalhos de linha onde a versão permitir. Aplicar em Filial > Vendedor > Cliente, Categoria > Subcategoria > SKU e Fornecedor > Categoria > Produto. Usar formatação condicional moderada, barras de dados, ícones de alerta e totais coerentes; validar subtotais e desempenho.
5. **Temas e Fluent 2:** explorar o tema Fluent 2 e o painel de personalização do relatório, caso presentes na versão. Criar e aplicar tema JSON TORK como fonte de verdade para cores, fontes e padrões visuais. Usar configurações globais para consistência, minimizando ajustes individuais; documentar as propriedades não controláveis por tema. Não substituir identidade da marca por tema padrão.
6. **Espaçamento dos gráficos:** ajustar margens internas e área de plotagem dos visuais nativos, conforme opções disponíveis, evitando rótulos cortados, legendas comprimidas e excesso de espaço vazio. Definir alinhamentos e espaçamentos consistentes em todas as páginas e testar em 16:9.

### Integração com as quatro páginas
- **Visão Executiva:** segmentadores modernos + calendário + rosca de mix com centro dinâmico + gráfico de evolução com área de plotagem bem distribuída.
- **Comercial & Clientes:** matriz expansível Cliente/Vendedor e segmentação RFV; alternância entre receita, margem, volume e clientes por field parameters, com slicers persistentes.
- **Estoque & Supply:** matriz hierárquica de SKU/filial, cabeçalhos fixos se disponíveis, rosca de situação de estoque e filtro de data do snapshot mais recente.
- **Rentabilidade & Ações:** matriz de rentabilidade com hierarquias, segmentadores compactos e alternância de análises Price/Volume/Mix, margem e oportunidades.

### Interações, HTML e compatibilidade
- Combinar **field parameters, bookmarks, botões e tooltips** para alternar visuais sem duplicação desnecessária de medidas. Configurar bookmarks para preservar ou redefinir filtros intencionalmente, conforme a interação.
- HTML/CSS/SVG devem complementar, não substituir sem necessidade, os visuais nativos modernizados. Animações discretas apenas onde o visual HTML efetivamente suportar o recurso; não presumir JavaScript, DOM irrestrito, eventos de clique em HTML ou integração direta com slicers.
- Criar **fallback nativo** para recursos em preview ou ausentes no ambiente. Testar Desktop, Service, exportação e acessibilidade; registrar evidências e diferenças.

### Critérios de aceite adicionais
- Entregar uma matriz de compatibilidade com: recurso, versão do Power BI, estado (GA/preview/indisponível), configuração utilizada, página, evidência de teste e fallback.
- Mostrar ao menos um uso funcional de cada recurso suportado, sem aumentar artificialmente a quantidade de visuais.
- Validar que filtros alteram corretamente roscas, matrizes, KPIs, HTML e parâmetros; que as hierarquias expandem sem alterar totais; e que os controles permanecem alinhados.
- **Não tratar a imagem como prova técnica de disponibilidade.** Consultar documentação oficial e verificar a instalação antes de implementar.

## RESULTADO VISUAL ALMEJADO
Um dashboard TORK com aparência de produto executivo premium, inspirada na identidade automotiva da marca, mas predominantemente **claro, legível e corporativo**. O diferencial não é quantidade de gráficos: é conseguir alternar análises sem poluir a tela, apresentar recomendações rastreáveis e transformar dados de vendas, clientes, estoque e margem em decisões operacionais.

**Comece agora pela Etapa A, lendo os arquivos reais. Não invente campos, resultados ou recursos de preview.**

---

# ADENDO OBRIGATÓRIO — UX PREMIUM, LANDING PAGE, FILTROS OCULTOS E TABELAS AVANÇADAS

**Este adendo prevalece sobre instruções anteriores quando houver divergência de layout. Implemente as regras abaixo em TODAS as páginas do projeto.** Não trate como sugestões opcionais.

## A. PÁGINA INICIAL (LANDING PAGE)
- Criar uma página inicial independente das quatro páginas analíticas, com o nome **Início**.
- Localizar na pasta do PBIP as imagens de fundo/templates fornecidos e utilizá-los como base visual, preservando proporção, resolução e legibilidade. Não gerar fundos genéricos se já existirem assets. Se houver múltiplas imagens, selecionar a mais apropriada e documentar a escolha.
- Aplicar logo TORK, título curto, subtítulo dinâmico quando houver contexto real e botão principal **INICIAR ANÁLISE**, grande, estilizado e funcional, direcionando para Visão Executiva por navegação de página ou bookmark.
- Manter composição limpa, com destaque para a marca e CTA, sem KPIs fictícios nem texto excessivo. Botão com estados hover/pressed quando suportados pelo visual nativo.

## B. FILTROS OCULTOS COM ABERTURA POR BOTÃO
- Em cada página analítica, criar painel lateral de filtros **oculto por padrão**, aberto pelo botão de ícone funil **FILTROS** e fechado pelo botão **X / APLICAR E FECHAR**. Usar **Selection Pane + Bookmarks + botões nativos**; garantir estados aberto/fechado, ordem de camadas, foco e consistência entre páginas.
- Configurar bookmarks de abertura/fechamento com **Data desativado**, sempre que adequado, para **não apagar nem substituir as seleções dos segmentadores**. Testar a persistência de filtros ao abrir/fechar e ao navegar.
- Dentro do painel, utilizar segmentadores modernos (filial, vendedor, categoria, cliente, período e filtros específicos da página), pesquisa quando pertinente, botão **Limpar filtros** com comportamento documentado e controles visualmente alinhados.
- Os segmentadores não devem permanecer espalhados pelo canvas principal. Deixar visível apenas o botão do painel e o resumo dinâmico dos filtros ativos.
- Quando útil, sincronizar segmentadores entre páginas; documentar quais são globais e quais são locais.

## C. BARRA SUPERIOR HTML — FILTROS ATIVOS
- Logo abaixo do cabeçalho, criar um **componente HTML/SVG personalizado, horizontal e responsivo**, que exiba os filtros efetivamente aplicados no contexto da página: período, filial, vendedor, categoria, cliente e outros relevantes.
- Renderizar filtros como chips/badges modernos com rótulos curtos, ícones SVG simples e cores TORK. Ex.: `PERÍODO: 2026 YTD` · `FILIAL: Campinas` · `CATEGORIA: Baterias` · `VENDEDOR: Todos`.
- Usar medidas DAX (`ISFILTERED`, `ISCROSSFILTERED`, `HASONEVALUE`, `SELECTEDVALUE`, `CONCATENATEX` com limite de itens, etc.) para produzir texto **verdadeiramente dinâmico**; diferenciar filtro direto de contexto cruzado quando necessário. Se houver múltiplos itens, mostrar `3 selecionados` em vez de truncar nomes incorretamente.
- Exibir estado padrão elegante (`Todas as filiais`, `Todas as categorias`), data de corte e indicação de período sintético futuro quando pertinente.
- **Não prometer clique em chips HTML para remover filtros**: HTML Content normalmente não oferece interação com segmentadores por eventos HTML/JavaScript. Para ações de limpar ou editar, fornecer **botões nativos do Power BI** próximos à barra, ou mecanismo comprovadamente suportado.
- Priorizar altura compacta e legibilidade em 16:9; não deixar chips quebrarem a composição.

## D. CABEÇALHOS E TÍTULOS DOS VISUAIS
- Criar cabeçalho principal de cada página em HTML/SVG premium: nome da página, subtítulo contextual calculado, detalhe de marca discreto, linha/acento vermelho e, quando fizer sentido, um microindicador contextual.
- **Centralizar os títulos de TODOS os visuais**, inclusive gráficos, tabelas, matrizes e blocos HTML. Usar título nativo centralizado quando disponível ou título HTML separado, mas nunca duplicar título.
- Subtítulos, comparações e mensagens devem ser dinâmicos com DAX e responder a filtros; não escrever conclusões fixas sobre os dados.
- Evitar animações chamativas ou contínuas. Usar microanimações CSS leves, condicionadas à compatibilidade do visual HTML e a `prefers-reduced-motion` quando possível. Garantir fallback estático.

## E. MATRIZES E TABELAS — PADRÃO VISUAL EXECUTIVO
- Priorizar **Matrix/Table nativas** para preservar ordenação, expansão, cross-filter, acessibilidade e exportação. Aplicar a todas as tabelas das quatro páginas.
- **Primeira coluna de dados:** valores **alinhados à esquerda**, com padding consistente. **Cabeçalho da primeira coluna centralizado**. **Cabeçalhos das demais colunas também centralizados**; valores numéricos preferencialmente alinhados à direita para comparação quantitativa, sem perder consistência estética.
- Se o visual nativo não permitir separar alinhamento do cabeçalho e do corpo por coluna, **não fingir que permite**: testar opções da versão instalada e registrar limitação/fallback (por exemplo, sobreposição de título/cabeçalho cuidadosamente construída, sem sacrificar ordenação).
- Aplicar tipografia legível, linhas discretas, cabeçalho destacado, zebra striping muito sutil, gridlines mínimas, espaçamento confortável, totais e subtotais elegantes, largura de colunas calibrada e hierarquia visual clara.
- Usar **data bars**, ícones de tendência, semáforos e formatação condicional baseada em medidas (margem, crescimento, gap, cobertura, risco), com cores consistentes e legenda acessível.
- Incorporar **minigráficos/sparklines nativos** nas células de matrizes/tabelas, quando suportados, por exemplo evolução mensal de faturamento por cliente, margem por vendedor, cobertura por SKU e pontualidade por fornecedor. Se o visual não aceitar o minigráfico no arranjo escolhido, usar barra de dados ou tooltip com tendência.
- Usar barras horizontais nas células para `% Meta`, `Margem %`, `Cobertura` e `Share estimado`, sem confundir percentuais com valores absolutos; sempre preservar valor textual.
- Evitar tabelas excessivamente largas, rolagem horizontal e excesso de colunas: mostrar 5–8 colunas principais e transferir detalhes para tooltip ou drillthrough. Garantir que linhas de total não exibam agregações sem sentido.
- Nas matrizes, testar hierarquias, expand/collapse, cabeçalhos fixos quando disponíveis e estados de drill; não perder alinhamento após expandir.

## F. TOOLTIP PAGES E TOOLTIP HTML/SVG
- Criar **tooltips de página de relatório** (report page tooltips) para visuais-chave; dentro delas, combinar HTML/SVG com visuais nativos compactos, conforme compatibilidade.
- Construir cartões com micrográficos SVG (sparkline, bullet, mini waterfall, barras de composição, evolução) baseados em medidas DAX e filtros do ponto selecionado. Não usar valores estáticos.
- Exemplos: **Cliente** → faturamento 12 meses, margem, frequência, mix, risco de churn; **SKU** → demanda, cobertura, curva ABC/XYZ, lead time; **Fornecedor** → OTIF, atraso, preço e evolução; **Filial/Vendedor** → meta x realizado, margem e tendência.
- Garantir que tooltips não cortem conteúdo, sejam rápidos, usem contraste adequado e tenham fallback de tooltip nativo. Validar que filtros e contexto da categoria chegam corretamente ao tooltip.
- Não presumir JavaScript, bibliotecas CDN, CSS global ou interações HTML fora do sandbox do visual utilizado.

## G. NAVEGAÇÃO ESTILIZADA E INTERAÇÃO ENTRE VISUAIS
- Criar navegação premium padronizada com **botões nativos** estilizados: Início, Visão Executiva, Comercial & Clientes, Estoque & Supply, Rentabilidade & Ações. Ícones vetoriais simples, estado ativo destacado, bordas arredondadas e hover/pressed quando suportado.
- Oferecer navegação por Page Navigator ou botões com ações, priorizando manutenção fácil e comportamento consistente. Usar botão **Voltar ao início**.
- Criar alternância entre visuais ou indicadores por **Field Parameters**, **bookmarks** ou **parâmetros desconectados + SWITCH DAX**: `Faturamento | Margem | Volume | Clientes`, `R$ | %`, `Mês | Trimestre | Ano`, `Ranking | Tendência | Participação` onde fizer sentido.
- Diferenciar **alternância de medida no mesmo visual** de **troca entre tipos de gráficos**: usar parâmetros de campo para a primeira e bookmarks/Selection Pane para a segunda. Configurar bookmarks para não sobrescrever filtros ativos inadvertidamente.
- Manter interação cruzada seletiva: cliques em gráficos devem filtrar/destacar outros visuais de forma útil; desabilitar interações confusas. Incluir drillthrough quando agregar valor.

## H. LAYOUT: POUCOS VISUAIS, ZERO BURACOS
- Construir cada página em **16:9**, com grid e margens padronizadas, respeitando fundos/templates existentes na pasta PBIP.
- Evitar superlotação: preferir **3–5 blocos analíticos principais por página**, além do cabeçalho, barra de filtros ativos, navegação e KPIs essenciais. Agrupar KPIs relacionados em uma faixa compacta.
- Não deixar **áreas vazias acidentais**; distribuir os visuais com equilíbrio, sem preencher cada pixel. **Espaço em branco intencional** para respiro visual é desejável. Não criar gráficos decorativos só para ocupar espaço.
- Usar cards, painéis e divisórias com raio, sombra e borda discretos; manter contraste, densidade e alinhamento consistentes. Priorizar identidade TORK (azul-marinho, vermelho, cinzas, off-white).
- Cada bloco deve responder a uma pergunta de negócio. Remover qualquer visual redundante ou que não gere decisão.
- Testar legibilidade em tela comum de notebook e exportação, sem elementos cortados, sobrepostos ou fora da área segura.

## I. RESPONSIVIDADE REALISTA DO HTML
- HTML/SVG deve usar `viewBox`, `preserveAspectRatio`, larguras relativas, `max-width`, flex/grid com fallback e tipografia escalável dentro das restrições do visual HTML utilizado.
- Testar componentes em pelo menos três tamanhos de container: compacto (KPI), médio (tooltip) e largo (cabeçalho/faixa de filtros). Evitar larguras fixas, overflow, scrollbars internos e dependência de media queries não suportadas.
- **Power BI Desktop não é um navegador responsivo geral**: não prometer reorganização automática de todo o canvas. Criar **layout mobile separado** se necessário e suportado, mantendo o canvas desktop bem resolvido.
- Componentes HTML não substituem botões nativos em ações que precisam filtrar ou navegar no relatório, salvo comprovação de suporte do visual específico.

## J. EXECUÇÃO E TESTES DE ACEITE — OBRIGATÓRIOS
1. Identificar imagens/templates reais da pasta PBIP e mostrar qual foi aplicado a cada página.
2. Implementar a landing page e testar o botão **INICIAR ANÁLISE**.
3. Implementar em **todas as páginas** os estados de painel de filtros oculto/aberto, testando se seleções persistem.
4. Implementar barra HTML de filtros ativos e testar seleção única, múltipla, nenhuma e filtros cruzados.
5. Confirmar título centralizado em cada visual e primeira coluna de tabela à esquerda com cabeçalhos centralizados (ou registrar limitação técnica real).
6. Demonstrar pelo menos uma matriz com sparkline e data bars funcionais, incluindo comportamento sob filtro e ordenação.
7. Implementar e testar ao menos quatro tooltips contextualizados: cliente, produto, fornecedor e filial/vendedor.
8. Validar navegação ativa, botão de início, alternância de medidas/tipos de gráficos e preservação dos filtros.
9. Auditar que **todos os textos de desempenho, insights, alertas e indicadores** dependem de DAX; textos estruturais como nomes de páginas e rótulos de botões podem ser estáticos.
10. Testar HTML em tamanhos diferentes, Desktop/Service quando disponíveis, além de exportação, performance e acessibilidade. Registrar screenshots ou evidências e pendências, sem inventar testes.

**Prioridade de implementação:** (1) modelo/medidas corretos → (2) landing page + templates → (3) navegação e painel de filtros → (4) barra HTML dinâmica → (5) componentes e tabelas → (6) tooltips e alternâncias → (7) QA final. **Execute o projeto, não apenas descreva como fazê-lo.**
