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
