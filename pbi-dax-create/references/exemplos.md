# Exemplos · /pbi-dax-create

Cookbook de medidas DAX comuns que servem como **referência rápida** ao gerar nova medida. Pra cada exemplo: padrão DAX recomendado + variações + quando usar.

> ⚠️ Esses exemplos usam nomes genéricos (`Vendas`, `Faturamento`, `Calendario`). Ao gerar a medida real, **substituir pelas tabelas/colunas/medidas que existem no projeto**.

---

## Categoria 1 · Métricas base (agregações)

### Soma simples
```dax
Total Quantidade = SUM(fVendas[Quantidade])
```

### Soma de produto entre colunas (SUMX)
```dax
Faturamento = SUMX(
    fVendas,
    fVendas[Quantidade] * fVendas[PrecoUnitario]
)
```

### Custos (mesmo padrão que faturamento)
```dax
Custos = SUMX(
    fVendas,
    fVendas[Quantidade] * fVendas[CustoUnitario]
)
```

### Diferença entre medidas
```dax
Margem Bruta = [Faturamento] - [Custos]
```

### Média
```dax
Ticket Médio = AVERAGEX(
    VALUES(fVendas[NotaFiscal]),
    CALCULATE([Faturamento])
)
```

### Mínimo / Máximo
```dax
Maior Venda Individual = MAX(fVendas[ValorItem])
```

---

## Categoria 2 · Contagens

### Linhas (com filtro contextual)
```dax
Total Vendas = COUNTROWS(fVendas)
```

### Distintos (sempre na chave inteira quando possível)
```dax
Clientes Únicos = DISTINCTCOUNT(Cliente[ClienteKey])

# Pra Notas Emitidas (NFe é o ID natural):
Notas Emitidas = DISTINCTCOUNT(fVendas[NFe])
```

### Contagem com condição
```dax
Pedidos Acima de 1000 =
CALCULATE(
    COUNTROWS(fVendas),
    fVendas[ValorItem] > 1000
)
```

---

## Categoria 3 · Percentuais e ratios

### Margem percentual (sempre DIVIDE)
```dax
% Margem Bruta = DIVIDE([Margem Bruta], [Faturamento])
```

### Participação (% sobre o total)
```dax
% Participação Vendas =
DIVIDE(
    [Faturamento],
    CALCULATE([Faturamento], REMOVEFILTERS(Produto))
)
```

### Conversão
```dax
% Conversão = DIVIDE([Pedidos], [Visitantes])
```

### Diferença em percentual
```dax
Δ % vs Meta = DIVIDE([Faturamento] - [Meta], [Meta])
```

---

## Categoria 4 · Time intelligence

> ⚠️ Pré-requisito: tabela calendário (`dCalendario`) marcada com `dataCategory: Time` e relacionada à fato pela coluna data.

### Ano Anterior (Last Year)
```dax
Faturamento LY =
CALCULATE(
    [Faturamento],
    SAMEPERIODLASTYEAR(dCalendario[Data])
)
```

### Variação Year over Year
```dax
% Faturamento YoY = DIVIDE(
    [Faturamento] - [Faturamento LY],
    [Faturamento LY]
)
```

### Mês Anterior (Last Month)
```dax
Faturamento Mês Anterior =
CALCULATE(
    [Faturamento],
    DATEADD(dCalendario[Data], -1, MONTH)
)
```

### Acumulado no Ano (YTD)
```dax
Faturamento YTD = TOTALYTD([Faturamento], dCalendario[Data])
```

### Acumulado no Mês (MTD)
```dax
Faturamento MTD = TOTALMTD([Faturamento], dCalendario[Data])
```

### Média móvel 3 meses
```dax
Média Móvel 3M =
CALCULATE(
    AVERAGEX(VALUES(dCalendario[Mês]), [Faturamento]),
    DATESINPERIOD(dCalendario[Data], MAX(dCalendario[Data]), -3, MONTH)
)
```

### Últimos 12 meses (LTM)
```dax
Faturamento LTM =
CALCULATE(
    [Faturamento],
    DATESINPERIOD(dCalendario[Data], MAX(dCalendario[Data]), -12, MONTH)
)
```

### Variação Mês contra Mês (MoM)
```dax
% Faturamento MoM = DIVIDE(
    [Faturamento] - [Faturamento Mês Anterior],
    [Faturamento Mês Anterior]
)
```

---

## Categoria 5 · Field Parameter (seleção dinâmica)

> Padrão pra dashboards onde o usuário escolhe qual métrica ver.

### Tabela auxiliar (criada via Field Parameter no PBI Desktop)
```tmdl
table auxAnalise
    column auxAnalise          // label visível
    column 'auxAnalise Campos' // referência à medida (isHidden)
    column 'auxAnalise Pedido' // ordem (0, 1, 2...) (isHidden)
```

### Lê a opção selecionada
```dax
ID Selecionado = SELECTEDVALUE(auxAnalise[auxAnalise Pedido])
```

### Retorna a medida correspondente
```dax
Medida Selecionada =
SWITCH(
    [ID Selecionado],
    0, [Faturamento],
    1, [Margem Bruta],
    2, [Notas Emitidas],
    [Faturamento]  // default
)
```

### Título dinâmico (label do gráfico)
```dax
Titulo Dinâmico = UPPER(VALUES(auxAnalise[auxAnalise]))
```

### Título com sufixo
```dax
Titulo Por Produto =
SWITCH(
    [ID Selecionado],
    0, "FATURAMENTO",
    1, "MARGEM BRUTA",
    2, "NOTAS EMITIDAS",
    "MÉTRICA"
) & " POR PRODUTO"
```

---

## Categoria 6 · Filtros e segmentação

### Top N (Top 10 produtos)
```dax
Top 10 Produtos =
CALCULATE(
    [Faturamento],
    TOPN(10, VALUES(Produto[Nome]), [Faturamento], DESC)
)
```

### Ranking
```dax
Ranking Vendedor =
RANKX(
    ALL(Vendedor[Nome]),
    [Faturamento],
    ,
    DESC,
    DENSE
)
```

### Filtro por categoria específica
```dax
Vendas Eletrônicos =
CALCULATE(
    [Faturamento],
    Produto[Categoria] = "Eletrônicos"
)
```

### Filtro por múltiplas categorias
```dax
Vendas Eletrônicos ou Móveis =
CALCULATE(
    [Faturamento],
    Produto[Categoria] IN { "Eletrônicos", "Móveis" }
)
```

### Excluir uma categoria
```dax
Vendas Sem Devolução =
CALCULATE(
    [Faturamento],
    fVendas[TipoOperacao] <> "Devolução"
)
```

### Filtros AND (múltiplas condições)
```dax
Vendas VIP no Sudeste =
CALCULATE(
    [Faturamento],
    Cliente[Tipo] = "VIP",
    Geografia[Regiao] = "Sudeste"
)
```

### Remover filtros (total geral mesmo com slicer)
```dax
Faturamento Total Sem Filtro =
CALCULATE(
    [Faturamento],
    REMOVEFILTERS()
)
```

---

## Categoria 7 · Lógica condicional

### IF simples
```dax
Status Margem = IF([% Margem Bruta] > 0.3, "Boa", "Ruim")
```

### SWITCH (substitui IF aninhado)
```dax
Faixa Cliente =
SWITCH(
    TRUE(),
    [Faturamento Cliente] > 1000000, "Premium",
    [Faturamento Cliente] > 100000, "Médio",
    [Faturamento Cliente] > 0, "Pequeno",
    "Inativo"
)
```

### Cor condicional (pra usar em formato condicional)
```dax
Cor Margem =
SWITCH(
    TRUE(),
    [% Margem Bruta] >= 0.4, "#10B981",  // verde
    [% Margem Bruta] >= 0.2, "#F59E0B",  // amarelo
    "#EF4444"                              // vermelho
)
```

---

## Categoria 8 · Variáveis (legibilidade)

### Pattern recomendado pra qualquer medida com 2+ passos

```dax
% Variação YoY com Detalhe =
VAR ValorAtual = [Faturamento]
VAR ValorLY = CALCULATE([Faturamento], SAMEPERIODLASTYEAR(dCalendario[Data]))
VAR Variacao = ValorAtual - ValorLY
VAR Pct = DIVIDE(Variacao, ValorLY)
RETURN
    IF(
        ISBLANK(ValorLY),
        BLANK(),
        Pct
    )
```

Mais legível que tudo aninhado em uma linha + permite debug fácil.

---

## Categoria 9 · Métricas customer (RFM básico)

### Recência (dias desde última compra)
```dax
Dias Desde Última Compra =
DATEDIFF(
    MAX(fVendas[Data]),
    TODAY(),
    DAY
)
```

### Frequência (número de pedidos do cliente)
```dax
Frequência Cliente = DISTINCTCOUNT(fVendas[NumeroPedido])
```

### Valor médio (ticket médio do cliente)
```dax
Ticket Médio Cliente =
DIVIDE(
    [Faturamento],
    [Frequência Cliente]
)
```

### LTV simplificado
```dax
LTV =
CALCULATE(
    [Faturamento],
    REMOVEFILTERS(dCalendario)  // soma todo o histórico
)
```

---

## Categoria 10 · Métricas avançadas

### Soma cumulativa (running total)
```dax
Faturamento Cumulativo =
CALCULATE(
    [Faturamento],
    FILTER(
        ALL(dCalendario[Data]),
        dCalendario[Data] <= MAX(dCalendario[Data])
    )
)
```

### Vendas que cresceram vs LY (clientes recurrent only)
```dax
Vendas Mantidas =
SUMX(
    VALUES(Cliente[ClienteKey]),
    VAR ValorAtual = [Faturamento]
    VAR ValorLY = CALCULATE([Faturamento], SAMEPERIODLASTYEAR(dCalendario[Data]))
    RETURN
        IF(NOT ISBLANK(ValorLY), ValorAtual)
)
```

### Pareto (Top 20% que faz 80%)
```dax
% Cumulativo Vendas =
VAR FatProduto = [Faturamento]
VAR TotalGeral = CALCULATE([Faturamento], REMOVEFILTERS(Produto))
VAR FatRank =
    CALCULATE(
        [Faturamento],
        FILTER(
            ALL(Produto[Nome]),
            [Faturamento] >= FatProduto
        )
    )
RETURN
    DIVIDE(FatRank, TotalGeral)
```

---

## Categoria 11 · Antipatterns a EVITAR

### ❌ Coluna calculada quando podia ser medida

❌ Errado:
```dax
// Coluna em fVendas (materializa em memória pra cada linha):
ValorItem = fVendas[Quantidade] * fVendas[PrecoUnitario]
```

✅ Certo (medida com SUMX):
```dax
Faturamento = SUMX(fVendas, fVendas[Quantidade] * fVendas[PrecoUnitario])
```

### ❌ DISTINCTCOUNT em string de alta cardinalidade

❌ Errado (CPF é string):
```dax
Clientes Únicos = DISTINCTCOUNT(Cliente[CPF])
```

✅ Certo (criar coluna inteira como key):
```dax
Clientes Únicos = DISTINCTCOUNT(Cliente[ClienteKey])
```

### ❌ Operador / em vez de DIVIDE
Já coberto na seção de padrões. **Sempre DIVIDE**.

### ❌ CALCULATE com filtro implícito sem operador

❌ Errado (interpretado como TRUE/FALSE — erro):
```dax
Vendas Brasil = CALCULATE([Faturamento], Geografia[Pais])
```

✅ Certo:
```dax
Vendas Brasil = CALCULATE([Faturamento], Geografia[Pais] = "Brasil")
```

### ❌ Iteradores em fato gigante sem pré-filtro

❌ Pode ser lento:
```dax
Média Itens =
AVERAGEX(fVendas, fVendas[Quantidade])  // itera 5M linhas
```

✅ Reduzir escopo antes:
```dax
Média Itens Mês Atual =
AVERAGEX(
    CALCULATETABLE(fVendas, dCalendario[Data] >= EDATE(TODAY(), -1)),
    fVendas[Quantidade]
)
```

---

## Como usar este cookbook

Ao gerar uma medida nova, a skill consulta esta lista pra:
1. Encontrar pattern equivalente ao pedido (categoria + cálculo)
2. Adaptar tabelas/colunas pros nomes reais do projeto
3. Aplicar nomenclatura conforme padrão do projeto
4. Adicionar variações (3 sugestões de nome)

Se o pedido não cai em nenhum desses padrões, gerar do zero seguindo as regras de `padroes.md`.

---

*Catálogo consolidado em 2026-04-26. Baseado em SQLBI patterns + casos comuns brasileiros.*
