# Padrões DAX + Naming · /pbi-dax-create

Boas práticas DAX que a skill **sempre aplica** ao gerar medida nova, e como detectar/seguir o padrão de nomenclatura existente no projeto.

---

## Parte 1 · Boas práticas DAX

### Divisão: SEMPRE `DIVIDE`, NUNCA `/`

❌ Ruim:
```dax
% Margem = [Margem Bruta] / [Faturamento]
```
→ retorna **Infinity** ou **NaN** quando denominador = 0. Visual mostra "∞", erro vermelho, ou número absurdo.

✅ Bom:
```dax
% Margem = DIVIDE([Margem Bruta], [Faturamento])
```
→ retorna BLANK se denominador = 0. Visual fica vazio (esperado).

Opcional 3º parâmetro (alternativa quando 0):
```dax
% Margem = DIVIDE([Margem Bruta], [Faturamento], 0)  // 0 em vez de BLANK
```

---

### Agregação: SUM vs SUMX

**SUM(tabela[col])** — quando soma DIRETA de UMA coluna:
```dax
Total Quantidade = SUM(fVendas[QtdItens])
```

**SUMX(tabela, expressão)** — quando expressão envolve MÚLTIPLAS colunas POR LINHA:
```dax
Faturamento = SUMX(
    fVendas,
    fVendas[QtdItens] * fVendas[PrecoUnitario]
)
```

**❌ Erro comum**: criar coluna calculada `[ValorLinha] = QtdItens × Preco` e fazer `SUM([ValorLinha])`. Funciona, mas **infla o .pbix** (materializa). SUMX é o caminho.

Mesma lógica vale pra `AVERAGEX`, `MAXX`, `MINX`, `RANKX`, `COUNTX`.

---

### Contagem: DISTINCTCOUNT vs COUNTROWS

**DISTINCTCOUNT(tabela[col])** — conta valores únicos numa coluna:
```dax
Clientes Únicos = DISTINCTCOUNT(Cliente[CPF])
```
⚠️ **Performance**: prefira coluna **inteira** quando possível (10× mais rápida que string).

**COUNTROWS(tabela)** — conta linhas (com filtros aplicados):
```dax
Total Vendas = COUNTROWS(fVendas)
```

**COUNTROWS(VALUES(col))** — alternativa equivalente a DISTINCTCOUNT, às vezes mais performática:
```dax
Clientes Únicos = COUNTROWS(VALUES(Cliente[CPF]))
```

---

### Time intelligence: SEMPRE referenciar coluna de calendário dedicada

Pré-requisito: ter tabela `dim Calendario` (ou similar) marcada com `dataCategory: Time` e relacionada à fato pela coluna data. Se o modelo não tem isso, sinalizar limitação.

**SAMEPERIODLASTYEAR** — mesmo período do ano passado (mais comum):
```dax
Vendas LY =
CALCULATE(
    [Faturamento],
    SAMEPERIODLASTYEAR(dCalendario[Data])
)
```

**DATEADD** — desloca data por X períodos:
```dax
Vendas Mês Anterior =
CALCULATE(
    [Faturamento],
    DATEADD(dCalendario[Data], -1, MONTH)
)
```

**DATESYTD / TOTALYTD** — acumulado no ano até a data:
```dax
Faturamento YTD = TOTALYTD([Faturamento], dCalendario[Data])
```

**% YoY** — variação ano contra ano (padrão DIVIDE):
```dax
% Vendas YoY = DIVIDE(
    [Faturamento] - [Vendas LY],
    [Vendas LY]
)
```

⚠️ Se o modelo tem **Auto Date/Time ligado** sem dCalendario marcada, time intelligence pode usar a hierarquia errada silenciosamente. **Sempre referenciar coluna explícita** da dim de tempo.

---

### CALCULATE: filter modifier explícito

❌ Ruim (regra confusa de contexto):
```dax
Vendas Brasil = CALCULATE([Faturamento], Geografia[Pais])
```
→ esse `Pais` sem operador é interpretado como `Geografia[Pais] = TRUE/FALSE` → erro.

✅ Bom (filtro explícito):
```dax
Vendas Brasil = CALCULATE(
    [Faturamento],
    Geografia[Pais] = "Brasil"
)
```

✅ Pra remover filtro:
```dax
Vendas Total Sem Filtro = CALCULATE(
    [Faturamento],
    REMOVEFILTERS(Geografia)
)
```

✅ Pra modificar direção de filtro:
```dax
Produtos Vendidos =
CALCULATE(
    DISTINCTCOUNT(Produto[ID]),
    CROSSFILTER(fVendas[cdProduto], Produto[ID], Both)
)
```

---

### CALCULATE aninhado: refatorar com VAR

❌ Ruim (CALCULATE dentro de CALCULATE):
```dax
Resultado = CALCULATE(
    [Faturamento],
    CALCULATE(
        VALUES(Cliente[Segmento]),
        Cliente[Tipo] = "VIP"
    )
)
```

✅ Bom (VAR + RETURN):
```dax
Resultado =
VAR SegmentosVIP =
    CALCULATETABLE(
        VALUES(Cliente[Segmento]),
        Cliente[Tipo] = "VIP"
    )
RETURN
    CALCULATE(
        [Faturamento],
        Cliente[Segmento] IN SegmentosVIP
    )
```

Ganho: legibilidade + debug fácil + DAX engine otimiza melhor.

---

### Variáveis: USE quando há 2+ passos

**Regra prática**: se a medida tem 2+ cálculos intermediários ou repete a mesma expressão, use VAR.

```dax
% Margem YoY =
VAR MargemAtual = [Margem Bruta]
VAR MargemLY = CALCULATE([Margem Bruta], SAMEPERIODLASTYEAR(dCalendario[Data]))
VAR Variacao = MargemAtual - MargemLY
RETURN
    DIVIDE(Variacao, MargemLY)
```

Mais legível que tudo aninhado em DIVIDE.

---

### IF / SWITCH

**IF** simples — 1 condição binária:
```dax
Status = IF([Faturamento] > 1000000, "Alto", "Baixo")
```

**SWITCH** — múltiplas condições (substitui IF aninhado):
```dax
Categoria Cliente =
SWITCH(
    TRUE(),
    [Faturamento] > 1000000, "Premium",
    [Faturamento] > 100000, "Médio",
    [Faturamento] > 0, "Pequeno",
    "Inativo"
)
```

❌ **Evitar**: `IF([Fat] > 1M, "Premium", IF([Fat] > 100K, "Médio", IF(...)))` → ilegível.

---

### Format string: usar quando faz sentido

```dax
measure 'Faturamento'
    formatString: "R$ #,0;-R$ #,0;R$ 0"
```

- **Moeda BR**: `"R$ #,0;-R$ #,0;R$ 0"`
- **Percentual 1 casa**: `"0.0%;-0.0%;0.0%"`
- **Inteiro com separador**: `"#,0"`
- **Decimal 2 casas**: `"#,0.00"`

---

## Parte 2 · Detecção e aplicação de naming convention

### Como detectar o padrão dominante

Ler todas as medidas existentes e classificar pelos padrões abaixo. O dominante (>50%) é o que a skill segue. Se há mistura sem dominante, sinalizar "modelo tem naming inconsistente — vou seguir o padrão SQLBI" (Title Case PT).

### Padrões comuns

| Padrão | Exemplos | Quando usar |
|---|---|---|
| **Title Case PT** | `Total Vendas`, `% Margem Bruta`, `Notas Emitidas` | Default recomendado SQLBI |
| **Title Case + prefixo tipo** | `% Faturamento YoY`, `# Pedidos`, `Δ MoM Vendas` | Quando múltiplas variantes da mesma medida |
| **camelCase** | `totalVendas`, `pctMargemBruta` | Time já usa essa convenção (raro em PBI BR) |
| **snake_case** | `total_vendas`, `margem_bruta_pct` | Time vindo de SQL/Python (raro em PBI BR) |
| **MAIÚSCULAS** | `TOTAL VENDAS`, `MARGEM BRUTA` | Pra títulos visuais (não pra medidas) |

### Sufixos de tipo (padrão SQLBI)

Quando há múltiplas variantes da mesma métrica:

| Sufixo | Significado | Exemplo |
|---|---|---|
| `LY` | Last Year (ano passado) | `Faturamento LY` |
| `YoY` | Year over Year (variação) | `% Faturamento YoY` |
| `MoM` | Month over Month | `% Vendas MoM` |
| `YTD` | Year to Date | `Faturamento YTD` |
| `MTD` | Month to Date | `Faturamento MTD` |
| `LTM` | Last Twelve Months | `Faturamento LTM` |
| `Δ` | Delta (variação absoluta) | `Δ Vendas vs LY` |

### Prefixos de tipo

| Prefixo | Significado | Exemplo |
|---|---|---|
| `%` | Percentual | `% Margem Bruta` |
| `#` | Contagem | `# Pedidos` |
| `Σ` | Soma agregada | `Σ Receita` (raro) |

### Geração das 3 sugestões

Sempre devolver 3 nomes em ordem de preferência:

1. **Recomendado** — segue 100% padrão dominante + tipo correto
2. **Alternativa formal** — variação mais explícita (mais palavras, menos abreviação)
3. **Alternativa enxuta** — variação mais curta (acrônimo se aplicável)

Exemplo: pedido "% margem bruta vs ano passado"
1. `% Margem Bruta YoY` (padrão Xperiun/SQLBI — recomendado)
2. `Variação Margem Bruta vs Ano Anterior` (mais explícito)
3. `Δ MB YoY` (mais curto, requer leitor saber MB = Margem Bruta)

---

## Parte 3 · DisplayFolder

Se o modelo já organiza medidas em displayFolders, sugerir folder existente que faça sentido. Padrões comuns:

- `0. Medidas Padrão` — base (Faturamento, Custos, Margem)
- `1. Medidas Temporais` — YoY, LY, YTD, MoM
- `2. Medidas por Categoria` — slicing por dim (Top N por Cliente, etc.)
- `3. Medidas Dinâmicas` — Field Parameter, IF/SWITCH dependente de seleção
- `4. Medidas Auxiliares` — internas, formatação, títulos

Se o modelo não tem displayFolders, **não inventar** — sugerir mas não forçar.

---

## Parte 4 · Description

Toda medida nova **deveria** ter `description:`. Padrão sugerido:

```tmdl
measure 'Faturamento'
    description: "Receita total das vendas (Quantidade × Preço Unitário). Em R$, sem descontos."
    formatString: "R$ #,0;-R$ #,0;R$ 0"
    = SUMX(fVendas, fVendas[QtdItens] * fVendas[PrecoUnitario])
```

Description ideal:
- **1-2 frases curtas** explicando o que calcula + unidade/contexto
- **Sem repetir o nome** ("Faturamento — calcula faturamento" → ruim)
- Em **PT-BR** com todos os acentos
- Mencionar **exclusões** importantes ("sem impostos", "sem devoluções", "apenas clientes ativos")

---

## Versão

Padrões consolidados em 2026-04-26. Baseado em SQLBI best practices + experiência Xperiun com modelos brasileiros.
