# Modelo relacional recomendado — Voltz 360°

## Estrela principal (filtros unidirecionais 1 → N)

```text
dCalendario[Data] ──1:N── fVendas[DataVenda]
dCliente[ClienteID] ──1:N── fVendas[ClienteID]
dProduto[ProdutoID] ──1:N── fVendas[ProdutoID]
dVendedor[VendedorID] ──1:N── fVendas[VendedorID]
dFilial[FilialID] ──1:N── fVendas[FilialID]

dCalendario[Data] ──1:N── fCompras[DataPedido]
dFornecedor[FornecedorID] ──1:N── fCompras[FornecedorID]
dProduto[ProdutoID] ──1:N── fCompras[ProdutoID]
dFilial[FilialID] ──1:N── fCompras[FilialID]

dCalendario[Data] ──1:N── fEstoque[Data]
dProduto[ProdutoID] ──1:N── fEstoque[ProdutoID]
dFilial[FilialID] ──1:N── fEstoque[FilialID]

dCalendario[Data] ──1:N── fFinanceiro[DataEmissao]
dCliente[ClienteID] ──1:N── fFinanceiro[ClienteID]
dFilial[FilialID] ──1:N── fFinanceiro[FilialID]

dCalendario[Data] ──1:N── fVisitas[DataVisita]
dCliente[ClienteID] ──1:N── fVisitas[ClienteID]
dVendedor[VendedorID] ──1:N── fVisitas[VendedorID]
dFilial[FilialID] ──1:N── fVisitas[FilialID]

dVendedor[VendedorID] ──1:N── fMetas[VendedorID]
dFilial[FilialID] ──1:N── fMetas[FilialID]
```

## Atenção à modelagem
- Não criar simultaneamente caminhos ativos dFilial → dVendedor → fVendas e dFilial → fVendas, nem dVendedor → dCliente → fVendas; desabilite relacionamentos entre dimensões para evitar ambiguidade.
- `fMetas[AnoMes]` deve se relacionar a uma dimensão mensal `dMes[AnoMes]` derivada de dCalendario; **não** ligar `fMetas[AnoMes]` diretamente a `dCalendario[AnoMes]` (lado não único).
- fVendas ↔ fFinanceiro: não ligar diretamente fatos em N:N. Se precisar de drillthrough por pedido, crie `dPedido` única a partir de PedidoID.
- Para datas alternativas de fCompras/fFinanceiro, usar USERELATIONSHIP ou dimensões calendário de papéis.
- O estoque é semanal: KPIs de saldo devem usar último snapshot no contexto, nunca somar saldos de várias semanas.
