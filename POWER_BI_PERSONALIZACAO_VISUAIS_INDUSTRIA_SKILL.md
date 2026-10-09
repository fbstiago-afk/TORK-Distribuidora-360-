# SKILL — PERSONALIZAÇÃO INTELIGENTE DE VISUAIS POWER BI

**Aplicação:** Torq Industrial | Indústria, manufatura, operações, produção, qualidade e supply chain  
**Uso:** documento de referência para Codex aplicar em um projeto Power BI/PBIP existente.  
**Escopo:** somente comportamento, estilo, formatação condicional e interação dos visuais. Não recriar o modelo nem inventar dados.

## 1. Princípio

Um gráfico deve responder visualmente **onde está a exceção, quanto ela representa e qual decisão merece atenção**. Usar recursos nativos do Power BI antes de HTML/SVG; reservar HTML Content para cartões ou componentes em que haja ganho real. Todos os cálculos devem respeitar contexto de filtro e granularidade.

### Referências
- LinkedIn: https://lnkd.in/p/dHn8sRF2 — sete padrões: crescimento/queda selecionável; rótulos personalizados; mínimo/máximo; abaixo da meta; meta vs realizado; Top N + Outros; quedas acima de limiar.
- YouTube: https://youtu.be/RaHi6yc02Tc — *Make this Creative & Insightful Line Chart in Power BI*. O título e o tema foram verificados; detalhes quadro a quadro do vídeo não foram confirmados. As técnicas de linhas abaixo são uma implementação proposta, não transcrição do tutorial.

## 2. Design system

- Preferir Modern Visual Defaults e tema centralizado, se disponíveis no ambiente.
- Cores: navy `#10263F`, azul `#2563EB`, teal `#0F9D92`, verde `#16A34A`, âmbar `#F59E0B`, vermelho `#DC2626`, cinza `#94A3B8`, fundo `#F5F7FA`.
- Usar cores semânticas, mas **inverter a interpretação quando menor é melhor** (scrap, PPM, atraso, custo).
- Manter fonte Segoe UI, margens consistentes, títulos orientados à pergunta, labels legíveis e tooltips de contexto.
- Não usar vermelho/verde como único meio de distinção: acrescentar ícone, texto ou posição.
- Evitar efeitos 3D e decoração que esconda valores.

## 3. Preparação e convenções

**Antes de editar:** inspecionar tabelas, medidas, relacionamentos, calendário, visuais existentes e artefatos PBIP. Criar cópia ou branch. Não supor nomes de tabelas.

Os exemplos usam `dCalendario[Data]`, `[Produção Real]`, `[Produção Meta]` e `[Refugo %]` como **placeholders**; mapear para as medidas reais. Marcar calendário como tabela de datas, validar continuidade e relacionamentos. Para metas, usar tabela de metas na granularidade correta de máquina, produto, planta, turno e período.

**Padrão de medidas:** `UX | ...` para auxiliares visuais; `KPI | ...` para indicadores; `Parâmetro | ...` para controles desconectados. Definir display folders no TMDL e descrições em português. Evitar medidas duplicadas.

## 4. Visual 01 — Linha com crescimento ou queda selecionável

**Objetivo:** ao escolher um mês, destacar a variação contra o mês anterior, mantendo a curva histórica visível.

**Aplicações:** produção mensal, OEE, volume de refugo, custo por unidade, lead time de fornecedor.

**Construção:**
1. Linha base em cinza/azul claro.
2. Slicer de período desconectado ou seleção de ponto, conforme suporte do visual.
3. Medida de referência do período anterior e delta absoluto/%.
4. Marcador e rótulo de destaque somente no período escolhido; tooltip com atual, anterior e diferença.
5. Mostrar seta e cor conforme direção **favorável ao KPI**, não apenas sinal matemático.

```DAX
UX | Produção Mês Anterior =
CALCULATE ( [Produção Real], DATEADD ( dCalendario[Data], -1, MONTH ) )

UX | Produção Variação % =
VAR Atual = [Produção Real]
VAR Anterior = [UX | Produção Mês Anterior]
RETURN IF ( ISBLANK ( Anterior ), BLANK (), DIVIDE ( Atual - Anterior, Anterior ) )
```

**Cuidado:** DATEADD requer calendário adequado e comparação de meses completos. Para mês corrente parcial, comparar janelas equivalentes ou sinalizar parcialidade.

## 5. Visual 02 — Rótulos personalizados com valor e variação

**Objetivo:** cada barra comunicar valor atual e variação YoY/MoM.

**Aplicações:** produção por linha, peças aprovadas por família, compras por fornecedor, perdas por causa.

**Padrão:** `12,4 mil un. | ▲ 8,2% vs AA` e tooltip com denominador. Quando o visual nativo não permitir concatenar rótulo de forma legível, usar tooltip ou visual compatível, sem forçar HTML.

```DAX
UX | Produção Ano Anterior =
CALCULATE ( [Produção Real], DATEADD ( dCalendario[Data], -1, YEAR ) )

UX | Produção YoY % =
VAR Anterior = [UX | Produção Ano Anterior]
RETURN IF ( ISBLANK ( Anterior ), BLANK (), DIVIDE ( [Produção Real] - Anterior, Anterior ) )

UX | Rótulo Produção =
VAR V = [Produção Real]
VAR P = [UX | Produção YoY %]
RETURN IF (
    ISBLANK ( V ), BLANK (),
    FORMAT ( V, "#,0" ) & " un. | " &
    IF ( ISBLANK ( P ), "sem comparação", IF ( P >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( P ), "0.0%" ) )
)
```

## 6. Visual 03 — Destaque automático de mínimo e máximo

**Objetivo:** o menor e o maior período se destacam mesmo quando o usuário troca ano, fábrica ou produto.

**Aplicações:** OEE por mês, disponibilidade por semana, FPY por turno, consumo energético por dia.

```DAX
UX | Cor Extremos Produção =
VAR Atual = [Produção Real]
VAR Periodos = ALLSELECTED ( dCalendario[AnoMes] )
VAR Minimo = MINX ( Periodos, CALCULATE ( [Produção Real] ) )
VAR Maximo = MAXX ( Periodos, CALCULATE ( [Produção Real] ) )
RETURN
SWITCH (
    TRUE (),
    ISBLANK ( Atual ), "#94A3B8",
    Atual = Minimo && Atual = Maximo, "#2563EB",
    Atual = Minimo, "#DC2626",
    Atual = Maximo, "#16A34A",
    "#94A3B8"
)
```

Aplicar como cor de dados via valor de campo quando suportado. Ajustar o domínio de `AnoMes` à dimensão do eixo. Empates recebem a mesma cor. **Mínimo não é sempre ruim:** inverter ou neutralizar conforme métrica.

## 7. Visual 04 — Valores abaixo da meta com limiar interativo

**Objetivo:** mudar a meta por parâmetro What-if e recolorir períodos/linhas abaixo do objetivo.

**Aplicações:** OEE mínimo, produção planejada, entregas no prazo, FPY.

```DAX
UX | Meta Ajustada =
COALESCE ( SELECTEDVALUE ( 'pMeta'[Valor] ), [Produção Meta] )

UX | Cor Atingimento =
VAR Real = [Produção Real]
VAR Meta = [UX | Meta Ajustada]
RETURN SWITCH (
    TRUE (),
    ISBLANK ( Real ) || ISBLANK ( Meta ), "#94A3B8",
    Real < Meta, "#DC2626",
    "#16A34A"
)
```

`pMeta` é tabela desconectada opcional; criar apenas se houver necessidade de simulação. Para refugo e atraso, aplicar regra **acima do teto = vermelho**.

## 8. Visual 05 — Meta vs realizado com leitura instantânea

**Objetivo:** visualizar gap sem precisar ler todos os números.

**Aplicações:** produção vs plano, OEE vs objetivo, manutenção preventiva executada vs programada, recebimentos vs demanda.

**Visuais preferidos:** bullet chart, barra com linha de meta, dumbbell ou colunas lado a lado. Destacar gap absoluto e percentual. Não usar gauge circular em série extensa.

```DAX
UX | Gap Produção = [Produção Real] - [Produção Meta]

UX | Atingimento Produção % =
DIVIDE ( [Produção Real], [Produção Meta] )
```

Distinguir `0`, `BLANK()` e ausência de meta. Atingimento acima de 100% não significa automaticamente qualidade ou eficiência melhores.

## 9. Visual 06 — Top N dinâmico + Outros

**Objetivo:** selecionar Top 5/10/15 e consolidar os demais em “Outros”, mantendo total reconciliado.

**Aplicações:** causas de parada, defeitos, fornecedores com atrasos, produtos com maior refugo, máquinas com perdas.

**Arquitetura recomendada:** tabela de parâmetro desconectada `pTopN`, dimensão de eixo desconectada com categorias e linha `Outros`, medida que soma Top N ou complemento do total. Não apenas aplicar filtro Top N no visual: isso **elimina** a categoria Outros.

```DAX
UX | N Selecionado = SELECTEDVALUE ( pTopN[N], 10 )
```

**Algoritmo:** `TOPN` sobre `ALLSELECTED` da dimensão, ordenar por medida e chave de desempate; `CONTAINS`/`TREATAS` para linhas individuais; `EXCEPT` + `SUMX` para Outros. Preservar filtros externos e verificar que Top N + Outros = total no contexto. Usar medida aditiva para ranking; para percentuais, agregar numerador/denominador antes de dividir.

## 10. Visual 07 — Quedas acima de limite definido pelo usuário

**Objetivo:** usuário seleciona limite de queda e o gráfico marca os dias/turnos com variação inferior ao limiar.

**Aplicações:** queda de produção, OEE, FPY, OTIF; para indicadores em que menor é melhor, monitorar **aumentos**.

```DAX
UX | Queda Crítica Produção =
VAR Variacao = [UX | Produção Variação %]
VAR Limite = SELECTEDVALUE ( pQueda[Percentual], 0.10 )
RETURN IF ( NOT ISBLANK ( Variacao ) && Variacao <= -Limite, 1, 0 )
```

`pQueda` é parâmetro desconectado com percentual decimal (ex.: 0,10). Para visual diário, criar medida de período anterior **diário** em vez de reutilizar a mensal. Aplicar pontos vermelhos e tooltip com causa candidata, sem declarar causalidade automaticamente.

## 11. Visual 08 — Gráfico de linha analítico premium

**Inspiração:** vídeo indicado. Implementação sugerida:
- Linha principal de espessura moderada com marcadores discretos.
- Último valor rotulado na ponta direita, se o visual permitir.
- Destaques separados para máximo, mínimo, último período e quebra de meta.
- Linha de referência de meta/média móvel.
- Faixa sombreada apenas quando representar intervalo de confiança ou banda operacional real.
- Tooltip com realizado, meta, período anterior, variação, média móvel e status.
- Parâmetro de campo para alternar Produção, OEE, Scrap e OTIF, respeitando escalas e unidades.
- Botões/bookmarks apenas se a interação for realmente útil.

**Média móvel de 3 meses (exemplo):**

```DAX
UX | Produção MM3 =
VAR Fim = MAX ( dCalendario[Data] )
VAR Janela = DATESINPERIOD ( dCalendario[Data], Fim, -3, MONTH )
VAR Meses = CALCULATETABLE ( VALUES ( dCalendario[AnoMes] ), Janela )
RETURN AVERAGEX ( Meses, CALCULATE ( [Produção Real] ) )
```

Validar a política para meses incompletos e a interação com filtros do calendário. Não ligar pontos sobre períodos sem dados como se fossem zero.

## 12. Adaptação por área industrial

| Área | Visual prioritário | Decisão |
|---|---|---|
| Operações | Linha OEE + meta + extremos | Identificar perda de eficiência |
| Operações | Quedas por turno + limiar | Investigar eventos críticos |
| Produção | Bullet Real vs Plano | Replanejar capacidade |
| Produção | Top N + Outros de paradas | Priorizar causas de perda |
| Qualidade | FPY com mínimo/máximo | Identificar turnos problemáticos |
| Qualidade | Refugo acima do limite | Abrir ação corretiva |
| Qualidade | Pareto de defeitos | Focar principais não conformidades |
| Supply | OTIF por fornecedor com meta | Cobrar entregas críticas |
| Supply | Lead time e desvios | Ajustar política de compras |
| Supply | Top N de materiais em risco | Priorizar abastecimento |

## 13. HTML + SVG: uso disciplinado

Empregar HTML Content apenas quando o visual nativo não fornecer a comunicação necessária. Exemplos: card com KPI, delta, barra de meta e sparkline; card de alerta operacional; resumo de exceções.

**Regras:**
- Gerar HTML por medida DAX somente com dados reais.
- Escapar textos provenientes de dados antes de inserir em markup; evitar HTML arbitrário.
- Usar SVG inline simples, sem scripts, recursos externos ou animações dependentes de JavaScript.
- Não presumir suporte a `<script>`, `setInterval` ou relógio em tempo real no Power BI Service.
- Evitar `FORMAT` em medidas numéricas de base: usar apenas em rótulos de apresentação.
- Testar renderização no Desktop e Service, responsividade, exportação e acessibilidade.

**Anatomia do card:** título → valor → delta → status → microtendência → referência da meta. Fundo claro, borda sutil, radius 12–18px, padding 16–20px, sem sombra pesada.

## 14. Interações e tooltips

- Tooltips devem responder: **quanto, comparado a quê, em qual período, em qual planta/linha e qual desvio?**
- Usar tooltip page quando precisar de série temporal, ranking ou decomposição.
- Sincronizar slicers somente onde fizer sentido; documentar filtros persistentes.
- Configurar edit interactions para evitar cross-highlights confusos.
- Drillthrough sugerido: planta → linha → máquina → turno → ordem, se as chaves existirem.
- Exibir estado de seleção, período parcial e ausência de dados.

## 15. Implementação em PBIP/TMDL

1. Ler `definition/` do modelo semântico e `definition/` do relatório no PBIP, quando presentes.
2. Localizar medidas existentes, formatos, display folders e dependências.
3. Criar medidas auxiliares em TMDL com descrições, sem renomear objetos usados por visuais sem migração.
4. Inspecionar estrutura real do PBIR antes de alterar `visual.json`/`page.json`; não inventar propriedades de formatação.
5. Aplicar tema global compatível com a versão instalada; preservar personalizações locais intencionais.
6. Validar DAX, referências, filtros, ordenação, tooltips, navegação e renderização.
7. Registrar mudanças em changelog e listar o que depende de configuração manual.

**Importante:** TMDL define o modelo semântico; a composição e a formatação de visuais pertencem ao relatório/PBIR e tema JSON. Não confundir esses formatos.

## 16. Regras de qualidade e performance

- Não criar visual baseado em métrica inexistente.
- Não confundir variação percentual com pontos percentuais.
- Não colorir toda queda de custos como negativa.
- Usar `ALLSELECTED` com intenção explícita; verificar subtotais e filtros.
- Evitar `FORMAT` como saída de medidas usadas em eixos ou ordenação.
- Para rankings e formatação condicional, verificar custo das medidas em visuais com muitas categorias.
- Comparar resultados com totais de controle e checar Top N + Outros.
- Medir desempenho no Performance Analyzer antes/depois.
- Documentar recursos Preview e fallback compatível.

## 17. Critérios de aceite

- [ ] Pelo menos uma exceção é identificável sem leitura minuciosa dos números.
- [ ] Cores têm significado consistente por indicador.
- [ ] Comparações usam períodos equivalentes e metas válidas.
- [ ] Parâmetros interativos atualizam medidas e formatação.
- [ ] Top N + Outros reconcilia com o total.
- [ ] Extremos respeitam os filtros ativos.
- [ ] Visuais permanecem legíveis em 16:9 e na resolução-alvo.
- [ ] Tooltips explicam contexto e não inventam causas.
- [ ] Sem dependência obrigatória de JavaScript em HTML Content.
- [ ] Alterações em TMDL, PBIR e tema foram verificadas separadamente.

## 18. Instrução curta para incorporar em outro prompt

> Consulte `POWER_BI_PERSONALIZACAO_VISUAIS_INDUSTRIA_SKILL.md` e aplique os padrões de personalização inteligente aos visuais existentes da Torq Industrial. Priorize destaques dinâmicos, metas, variações, Top N + Outros, limiares ajustáveis, rótulos contextuais e linhas analíticas. Preserve modelo, identidade visual e métricas já existentes; implemente apenas recursos compatíveis e relevantes. Valide medidas DAX, PBIR, TMDL, filtros, desempenho e consistência dos resultados.
