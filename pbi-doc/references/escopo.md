# Escopo · /pbi-doc

Define **o que vai em cada arquivo da documentação**. Esse documento é a "fonte da verdade" sobre quais campos, em qual ordem, com qual nível de detalhe — pra cada arquivo gerado.

---

## `00-overview.md` — Sumário do modelo

**Propósito**: leitor abre, entende o modelo em 30 segundos.

**Conteúdo**:
1. **Cabeçalho**
   - Nome do projeto (`.pbip`)
   - Data/hora geração
   - Tagline 1-linha do propósito (inferido a partir das tabelas: "modelo de vendas com análise temporal YoY", "modelo financeiro com DRE consolidado", etc.)

2. **Métricas-resumo**
   - N tabelas reais (excluindo auto-date)
   - N medidas
   - N relacionamentos
   - N colunas totais
   - Tamanho do .pbip (estimado pela soma dos .tmdl)

3. **Inventário de tabelas** (1 linha por tabela)
   - Tabela | Tipo (Fato / Dimensão / Medidas / Aux) | N colunas | N medidas hospedadas | Source resumido

4. **Fontes de dados** (parsing das partições M)
   - Lista única de fontes (Excel, SQL, Web, Sharepoint, etc.)
   - Path/conexão resumida
   - **Sinalizar paths pessoais** (Google Drive, OneDrive, C:\Users\) com aviso visual

5. **Configurações relevantes do modelo**
   - Auto Date/Time (on/off)
   - Culture (pt-BR/en-US/...)
   - Compatibility level
   - Outras flags importantes

---

## `01-tabelas.md` — Catálogo de tabelas

**Propósito**: pra cada tabela do modelo, descreve papel + colunas tipadas.

**Conteúdo (por tabela)**:

```markdown
## {Nome da tabela}

> {Tagline em 1 linha — papel da tabela no modelo, granularidade}
> Tipo: {Fato | Dimensão | Tabela de medidas | Auxiliar}
> Origem: {fonte resumida — ex: Vendas.xlsx (PlanilhaVendas) via Excel.Workbook}

### Descrição
{1-2 parágrafos. Se a tabela tem `description:` declarado no TMDL, usar. Senão, inferir do nome + colunas + uso em medidas.}

### Granularidade
{1 linha = ?  — ex: "1 linha por item de NFe", "1 linha por dia"}

### Colunas

| Coluna | Tipo | Papel | Notas |
|---|---|---|---|
| `cdProduto` | int64 | Chave estrangeira → FotoProduto | — |
| `Data` | dateTime | Data da venda | — |
| ... |

**Papel** = um de: Chave primária / Chave estrangeira / Atributo / Métrica / Calculada
**Notas** = sinalizar coisas relevantes: oculta, calculada, com formato especial, etc.

### Medidas hospedadas (se for "tabela de medidas")
- {Lista linkada pras medidas em 02-medidas.md, agrupadas por displayFolder}

### Source M (resumo)
```m
let Fonte = ...
in #"...":
```
{Não copiar o M inteiro se for >15 linhas — resumir os passos relevantes}
```

**Ordem**: tabelas-fato primeiro, depois dimensões, depois auxiliares (parameter tables, measure tables).

---

## `02-medidas.md` — Catálogo de medidas

**Propósito**: pra cada medida, mostra DAX original + explicação PT.

**Estrutura**: agrupar por **displayFolder** (que existe no TMDL). Se medida não tem displayFolder, agrupar em "Sem pasta".

**Conteúdo (por medida)**:

```markdown
### {Nome da medida}

`{tabela host}.{medida}` · {formatString se relevante} · {displayFolder}

**O que faz:**
{1-3 frases em PT explicando o resultado da medida sem entrar em DAX. Foco no business meaning.}

**DAX:**
\`\`\`dax
{DAX original, formatado, sem comentários originais alterados}
\`\`\`

**Como funciona:**
{Explicação técnica em PT da lógica DAX. Se medida usa outras medidas, listar quais. Se usa funções time intelligence, explicar o contexto. Se tem CALCULATE, explicar o filter modifier.}

**Usa:** {lista de outras medidas/colunas referenciadas}
**É usada por:** {lista de medidas que dependem desta — preencher após processar todas}
```

**Ordem dentro de cada displayFolder**: alfabética.

**Tom**: explicação em PT deve ser **didática mas não condescendente**. Pra analista que sabe DAX, mas pode não conhecer o modelo específico.

---

## `03-relacionamentos.md` — Mapa de relacionamentos

**Propósito**: visualizar quem se relaciona com quem, com que cardinalidade.

**Conteúdo**:

### Diagrama ASCII (texto)
```
                    ┌─────────────┐
                    │ dCalendario │
                    └──────┬──────┘
                           │ 1:N
                           ▼
       ┌──────────────┐  N  ┌─────────┐  N  ┌──────────────┐
       │ FotoVendedor │◄────│ fVendas │────►│ FotoProduto  │
       └──────────────┘     └─────────┘     └──────────────┘
                                    (bi-direcional ⚠)
```

(Se modelo tem >10 tabelas, simplificar pra showing só fato + dims principais.)

### Tabela detalhada de relacionamentos

| # | From | To | Cardinalidade | Direção | Ativo | Notas |
|---|---|---|---|---|---|---|
| 1 | fVendas.cdProduto | FotoProduto.'Cod Produto' | N:1 | **Bothdirections** | ✓ | Bi-direcional |
| 2 | fVendas.Data | dCalendario.Data | N:1 | Single | ✓ | — |

**Notas** = sinalizar bi-direcionais, inativos, M:M, etc.

### Análise rápida (1 parágrafo)
{Descrição em PT: "Modelo segue star schema com dCalendario como dimensão de tempo central. Apenas 1 fato (fVendas), 3 dimensões (Calendario, Produto, Vendedor). Único relacionamento bi-direcional é entre fVendas e FotoProduto — pode ser revisitado." Sem opinar (essa é função do review), só descrever.}

---

## `04-dependencias.md` — Grafo de dependências

**Propósito**: mostrar quem depende de quem entre as medidas. Ajuda no impacto de mudanças.

**Conteúdo**:

### Árvore por medida-raiz

Identificar **medidas-raiz** (que não são usadas por nenhuma outra) e mostrar a árvore descendente.

```
% Faturamento YoY
└─ Faturamento
│  └─ fVendas[QtdItens]
│  └─ fVendas[PrecoUnitario]
└─ Referência Faturamento LY
   └─ Faturamento (já mapeada acima)
   └─ dCalendario[Data]
```

### Lista reverse (impacto)

Pra cada medida **base** (usada por outras), listar quem depende.

```markdown
### Faturamento (base)

**Usada por:**
- Margem Bruta
- % Faturamento YoY
- Referência Faturamento LY
- Medida Selecionada

**Implicação:** mudar `Faturamento` afeta 4 outras medidas. Cuidado em refator.
```

### Tabelas mais referenciadas

Top 5 tabelas mais usadas em medidas (sinaliza onde mora a "carne" do modelo).

---

## Regras transversais

**Tom**: PT-BR direto, sem jargão desnecessário, com personalidade Xperiun (provocativo quando faz sentido, mas em doc é mais sóbrio que em /pbi-modelo-review).

**Acentuação**: SEMPRE com todos os acentos (regra inviolável CLAUDE.md).

**Excluir auto-date**: tabelas `LocalDateTable_*` e `DateTableTemplate_*` **não entram** em nenhum dos 5 arquivos. Se modelo tem essas tabelas, mencionar **só** no overview ("o modelo tem Auto Date/Time ligado, gerando 2 tabelas-fantasma ocultas — para auditar isso, rode `/pbi-modelo-review`").

**Linkar entre arquivos**: usar links markdown relativos. Ex: em `02-medidas.md`, ao mencionar uma tabela, linkar pra `01-tabelas.md#nome-tabela`.

**Code blocks DAX**: usar fence ` ```dax ` pra sintaxe Markdown highlight (e o HTML aplica syntax highlight via classes `.k`, `.f`, `.s`, `.c`).

**Não inventar números**: se modelo tem 4.2M linhas em fVendas, isso vem do partition info — não inventar tamanho de dados. Se não há info, não citar.

**Não opinar**: a `/pbi-doc` descreve, não julga. Opinião é da `/pbi-modelo-review`.


---

## Placeholders do `templates/relatorio.html`

O template HTML usa estes placeholders `{{...}}` que devem ser substituídos com valores reais derivados dos `.tmdl`. Substituir SOMENTE no HTML — nunca dentro de comentários `<!-- -->` (CSS, JS, comentários ficam intocados).

### Placeholders globais

| Placeholder | Conteúdo |
|---|---|
| `{{PROJECT_NAME}}` | Nome do projeto (ex: `16 - EV16 - Dashboard Vendas`) |
| `{{PROJECT_FILENAME}}` | Nome do arquivo `.pbip` |
| `{{TIMESTAMP}}` | Data de geração (ex: `26 abr 2026`) |
| `{{PROJECT_TAGLINE}}` | 1 frase descrevendo o propósito do modelo (inferido) |
| `{{PROJECT_HERO_SUB}}` | Subtítulo do hero (ex: `EV16 Power BI Week · Aula 01 · gerado em 26 abr 2026`) |
| `{{TABLES_COUNT}}`, `{{MEASURES_COUNT}}`, `{{RELATIONSHIPS_COUNT}}`, `{{COLUMNS_COUNT}}`, `{{SIZE}}` | Métricas inteiras |

### Blocos HTML (gerados pelo Claude com base nos `.tmdl`)

| Placeholder | Conteúdo |
|---|---|
| `{{NAV_TABLES_HTML}}` | Sub-nav de tabelas (`<a>` com badges fato/dim/med/aux) |
| `{{NAV_MEASURES_HTML}}` | Sub-nav de pastas de medidas (`<a>` com counts) |
| `{{INVENTORY_TABLE_ROWS}}` | Linhas `<tr>` da tabela inventário |
| `{{DATA_SOURCES_TEXT}}` | Texto descritivo das fontes |
| `{{WARNINGS_HTML}}` | Callouts.warn pra problemas detectáveis (paths pessoais, etc.) — pode ser vazio |
| `{{CONFIG_LIST_HTML}}` | Items `<li>` da config-list (culture, compatibility, autoDateTime, etc.) |
| `{{TABLES_CARDS_HTML}}` | Todos os `<article class="table-card">` da seção 01 |
| `{{MEASURE_GROUPS_HTML}}` | Todos os `<div class="measure-group">` da seção 02 |
| `{{REL_SVG_HTML}}` | SVG inline do diagrama de relacionamentos (gerar dinâmico) |
| `{{REL_TABLE_ROWS}}` | Linhas `<tr>` da tabela de relacionamentos |
| `{{DEP_TREE_HTML}}` | Árvore de dependências (uma ou mais) |
| `{{DEP_REVERSE_HTML}}` | Cards reverse das medidas-base |
| `{{TOP_TABLES_LIST_HTML}}` | Items `<li>` com tabelas mais referenciadas |

### Padrão de cada `<details class="measure-mini">`

Todas as medidas seguem este shape — medidas-âncora têm classe `.anchor` + atributo `open`:

```html
<details class="measure-mini [anchor]" [open]>
  <summary>
    <span class="name">{Nome}</span>
    <span class="dax">{DAX-essência em 1 linha}</span>
    <span class="toggle">+</span>
  </summary>
  <div class="mini-body">
    <p class="meta">Tabela: <code>{tabela}</code> · Format: <code>{format}</code></p>
    <p class="desc"><strong>O que faz:</strong> {explicação business}</p>
    <pre class="code">{DAX completo com syntax highlight via spans .k .f .s .c}</pre>
    <div class="label">Como funciona</div>
    <p class="desc">{explicação técnica}</p>
    <div class="label">É usada por</div>
    <p class="measure-deps"><code>{outra-medida-1}</code> · <code>{outra-medida-2}</code></p>
  </div>
</details>
```

### Excluir tabelas auto-date

`LocalDateTable_*` e `DateTableTemplate_*` são auto-geradas pelo Power BI quando Auto Date/Time está ON — **não fazem parte da doc intencional**. Skipar.
