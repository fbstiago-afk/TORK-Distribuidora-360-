# Voltz Distribuição Automotiva — Base sintética Distribuidora 360°

**Uso:** portfólio Power BI. Todos os clientes, fornecedores, valores e operações são fictícios.
**Período:** 2024-01-01 a 2026-10-31 (inclui datas futuras em relação à geração em 2026-10-08: são projeções sintéticas, não dados realizados).
**Formato:** CSV UTF-8, vírgula, datas ISO 8601, ponto decimal.

## Tabelas e volumes
- `dCalendario`: 1,035 registros
- `dCliente`: 2,000 registros
- `dFilial`: 4 registros
- `dFornecedor`: 50 registros
- `dProduto`: 700 registros
- `dVendedor`: 20 registros
- `fCompras`: 75,000 registros
- `fEstoque`: 414,400 registros
- `fFinanceiro`: 720,000 registros
- `fMetas`: 680 registros
- `fVendas`: 720,000 registros
- `fVisitas`: 48,000 registros

## Grãos, chaves e relacionamentos
- `dCalendario`: uma linha por dia; PK Data. Datas das fatos ligam a Data (papéis alternativos por relações inativas ou dimensões de data de papel).
- `dFilial`: PK FilialID; todas as fatos comerciais/operacionais têm FK FilialID.
- `dCliente`: PK ClienteID; FKs VendedorID, FilialID.
- `dVendedor`: PK VendedorID; FK FilialID.
- `dProduto`: PK ProdutoID; FK FornecedorPrincipalID.
- `dFornecedor`: PK FornecedorID.
- `fVendas`: PK VendaID, grão item de pedido; PedidoID compartilhado por linhas de mesmo cliente e data. FKs ClienteID, ProdutoID, VendedorID, FilialID, DataVenda.
- `fFinanceiro`: PK TituloID, grão título por PedidoID; FKs ClienteID, FilialID, DataEmissao. Um título por pedido, somatório igual à receita líquida das linhas.
- `fCompras`: PK CompraID, grão item de pedido de compra; FKs FornecedorID, ProdutoID, FilialID, DataPedido.
- `fEstoque`: PK composta (Data, FilialID, ProdutoID), snapshot SEMANAL aos sábados. Entrada=quantidades recebidas de compras até a data da semana; Saida=quantidades vendidas na semana. EstoqueFinal = EstoqueInicial + Entrada - Saida + Ajuste. Ajuste positivo = suprimento emergencial não rastreado em fCompras; não interpretar como compra.
- `fMetas`: PK composta (AnoMes, VendedorID), grão mês/vendedor; FilialID também disponível.
- `fVisitas`: PK VisitaID, grão interação comercial, FKs ClienteID, VendedorID, FilialID, DataVisita.

## Regras de negócio
- ReceitaBruta = Quantidade × PrecoTabelaUnitario (diferenças de centavos possíveis por arredondamento de campos). ReceitaLiquida = Quantidade × PrecoVendaUnitario. CMV = Quantidade × CustoUnitario. MargemContribuicao = ReceitaLiquida − CMV − FreteRateado − Comissao.
- fFinanceiro registra recebíveis; DataPagamento vazia em títulos não integralmente pagos. DiasAtraso apurado na data final da simulação para títulos pendentes.
- `PotencialMensalEstimado` é estimativa sintética, NÃO é dado observado de mercado; Share of Wallet é estimado.
- ABCBase é classificação inicial ilustrativa: para ABC real, recalcular com faturamento do período. XYZ deve ser calculada por variabilidade da demanda no Power BI.
- Metas são parâmetros comerciais sintéticos, não garantem atingimento.
- Feriados do calendário representam apenas uma lista simplificada de feriados fixos nacionais; não incluem móveis/municipais.
- Compras pendentes não entram no estoque; recebimentos fora do horizonte ficam pendentes.
- Datas de estoque são semanas encerradas aos sábados, não snapshots diários; a última semana encerrada é 2026-10-31.

## Eventos simulados e oportunidades analíticas
- Aumento de custos de baterias a partir de 2026, com pressão na margem e descontos adicionais no período.
- Fornecedor 3 com piora de atraso a partir do segundo semestre da série.
- Fornecedor 8 com score de entrega alto e score de preço baixo.
- Clientes com perfis de queda, inatividade, reativação e alto potencial.
- Clientes iniciais mais concentrados em faturamento e descontos maiores.
- Estoque com ajustes emergenciais, risco de cobertura e excesso em itens lentos.
- Vendas com sazonalidade, dias úteis, finais de mês e crescimento por ano.

## Análises sugeridas
1. Executivo: atual x ano anterior x meta, margem e projeção.
2. Comercial: RFV, churn, cross-sell, gap de potencial e concentração.
3. Supply: cobertura, giro, ABC/XYZ, compras abertas, OTIF, lead time e ruptura projetada.
4. Rentabilidade: waterfall de margem, preço-volume-mix, ranking por contribuição e centro de decisões.

## Limitações importantes
- Trata-se de dados **sintéticos**, não de extração ERP.
- Pedidos de vendas foram construídos a partir de combinações cliente/data/bloco; podem ter somente um item.
- Os ajustes emergenciais do estoque preservam saldos não negativos e devem ser analisados como exceções operacionais.
- Visitas simuladas não têm vínculo causal garantido com PedidoID; avaliar conversão com janela temporal, não atribuição direta.
- O arquivo fEstoque é semanal, apesar de vendas/compras serem diárias.
- Os dados não representam indicadores financeiros contábeis completos (não há despesas fixas/EBITDA auditável).
