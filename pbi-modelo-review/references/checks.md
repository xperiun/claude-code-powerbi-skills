# Checks · /pbi-modelo-review

Spec completa dos checks rodados pela skill. Cada check tem:
- **ID** (slug único)
- **Categoria** (uma das 6 famílias)
- **Severidade** (`critical` | `medium` | `light`)
- **Como detectar** (heurística aplicada nos `.tmdl`)
- **Por que importa** (vai pro relatório, tom Xperiun)
- **Como corrigir** (recomendação prática + snippet quando faz sentido)

---

## Família 1 · MODELAGEM

### `model-flat-detected`
- **Severidade**: `critical`
- **Como detectar**: contar tabelas que têm colunas tipicamente "transacionais" (data/timestamp + valor numérico + chave estrangeira) sem ter dimensões compartilhadas (Calendário, Produto, Cliente). Se ≥ 4 tabelas-fato candidatas e zero dimensões claras → modelo plano.
- **Por que importa**: 8 tabelas-fato sem dimensões compartilhadas força `RELATED/RELATEDTABLE` em quase toda medida, mata performance no DirectQuery e impossibilita filtros cruzados eficientes. Modelo plano é o anti-pattern #1 em Power BI corporativo.
- **Como corrigir**: identificar dimensões conformadas (Calendário, Produto, Cliente, Vendedor) e refatorar pra star schema clássico. Começar pelo Calendário — 90% das medidas time-intelligence dependem dele.
- **Snippet**:
```tmdl
table 'dim Calendario'
    dataCategory: Time
    isHidden: false

    column Data
        dataType: dateTime
        isKey: true
```

### `model-circular-reference`
- **Severidade**: `critical`
- **Como detectar**: parsing dos relacionamentos em `relationships.tmdl` — montar grafo direcionado e detectar ciclos.
- **Por que importa**: ciclos no grafo de relacionamentos quebram o motor de filtros — Power BI desativa relacionamentos automaticamente quando detecta, mas isso geralmente passa despercebido até alguém perguntar "por que esse total tá errado?".
- **Como corrigir**: identificar o relacionamento "extra" no ciclo, marcar como inativo (`isActive: false`) e usar `USERELATIONSHIP()` em medidas específicas que precisam dele.

### `model-no-date-table`
- **Severidade**: `critical`
- **Como detectar**: nenhuma tabela com `dataCategory: Time` E nenhuma tabela cujo nome contenha "calendar"/"calendário"/"data" com coluna do tipo `dateTime` marcada como `isKey: true`.
- **Por que importa**: sem tabela calendário marcada, time intelligence (`SAMEPERIODLASTYEAR`, `DATESYTD`, `TOTALYTD`) não funciona corretamente. Vira fonte de bugs sutis em medidas YoY/MTD/YTD.
- **Como corrigir**: criar tabela calendário dedicada (mesmo se já existe coluna data em fato — deve haver dimensão separada) e marcar com `dataCategory: Time`.

### `model-fact-as-dim`
- **Severidade**: `medium`
- **Como detectar**: tabela com nome contendo "vendas"/"pedidos"/"transacoes" sendo referenciada como lado "1" em relacionamento (deveria ser sempre o lado "N" como fato).
- **Por que importa**: confunde a leitura do modelo, indica modelagem inversa que vai gerar contagens erradas.
- **Como corrigir**: revisar cardinalidade do relacionamento — fato sempre N, dimensão sempre 1.

### `model-calendar-using-auto-hierarchy`
- **Severidade**: `critical`
- **Como detectar**: tabela calendário própria (com nome contendo "calendar"/"calendário"/"data" ou marcada com `dataCategory: Time`) tem coluna com `variation Variation` apontando `defaultHierarchy:` pra alguma `LocalDateTable_*` ou `DateTableTemplate_*` (auto-gerada pelo Auto Date/Time).
- **Por que importa**: pior dos mundos — você pagou o trabalho de criar calendário próprio (`dCalendario`) E continua pagando o custo das tabelas auto-geradas, **e ainda usa a errada como hierarquia padrão**. Medidas time intelligence (`SAMEPERIODLASTYEAR`, `DATESYTD`) podem cair na hierarquia auto sem você perceber. É a manifestação combinada do auto-date-time ligado + calendário não marcado como Time.
- **Como corrigir**:
  1. Marcar a tabela calendário com `dataCategory: Time`
  2. Marcar a coluna Data com `isKey: true`
  3. Remover a `variation Variation` da coluna (que aponta pra LocalDateTable)
  4. Desligar Auto Date/Time globalmente nas opções do Power BI Desktop
- **Snippet**:
```tmdl
table dCalendario
    dataCategory: Time
    isHidden: false

    column Data
        dataType: dateTime
        isKey: true
        // sem 'variation Variation' — usa a tabela própria, não a auto
```

---

## Família 2 · RELACIONAMENTOS

### `rel-bidirectional-unnecessary`
- **Severidade**: `critical`
- **Como detectar**: relacionamento com `crossFilteringBehavior: bothDirections` entre tabela claramente fato e tabela claramente dimensão (cardinalidade N:1).
- **Por que importa**: bi-direcional em N:1 com calendário/produto/cliente causa ambiguidade de filtro e duplica scan no engine. Em modelo de 1.4GB você está pagando ~30% de overhead por refresh sem ganhar nada.
- **Como corrigir**: mudar pra single direction. Se precisa filtrar dimensão a partir de fato em medida específica, usa `CROSSFILTER()` dentro do `CALCULATE` — controle cirúrgico, sem custo permanente.
- **Snippet**:
```dax
Vendas Filtradas =
CALCULATE(
    [Total Vendas],
    CROSSFILTER( Calendario[Data], Vendas[Data], Both )
)
```

### `rel-many-to-many-without-bridge`
- **Severidade**: `critical`
- **Como detectar**: relacionamento declarado com `cardinality: manyToMany` sem tabela bridge intermediária identificável.
- **Por que importa**: M:M direto sem bridge causa explosão de cardinalidade no engine, performance cai exponencialmente conforme dados crescem.
- **Como corrigir**: criar tabela bridge contendo as chaves únicas das duas pontas, refatorar pra dois N:1.

### `rel-inactive-orphan`
- **Severidade**: `light`
- **Como detectar**: relacionamentos com `isActive: false` que **não são referenciados** por nenhuma medida via `USERELATIONSHIP()`.
- **Por que importa**: relacionamento inativo não usado polui o modelo e confunde quem vier depois — "isso aqui era usado pra algo, posso tirar?".
- **Como corrigir**: remover relacionamentos inativos órfãos. Se foram criados "pro futuro", documentar a intenção.

### `rel-one-to-one-suspicious`
- **Severidade**: `medium`
- **Como detectar**: relacionamento `cardinality: oneToOne` entre duas tabelas.
- **Por que importa**: 1:1 quase sempre indica que as duas tabelas deveriam ser uma só. Cria join desnecessário em cada query.
- **Como corrigir**: avaliar se merge das tabelas resolve. Exceção legítima: tabela "extensão" com colunas pesadas que nem sempre carregam (ex: imagens, blobs).

### `rel-snowflake-suspicious`
- **Severidade**: `medium`
- **Como detectar**: cadeia de relacionamentos dim → dim → fato (ex: SubCategoria → Categoria → Produto → Vendas).
- **Por que importa**: snowflake quebra o princípio star schema — múltiplos joins por query, mais lento que dimensão denormalizada.
- **Como corrigir**: denormalizar dimensão (trazer atributos da SubCategoria pra dentro da Categoria, ou direto no Produto). Star schema vence quase sempre.

---

## Família 3 · PERFORMANCE

### `perf-calc-column-should-be-measure`
- **Severidade**: `critical`
- **Como detectar**: coluna calculada em tabela-fato cuja expressão não usa `EARLIER`, `RANKX`, ou agregação iterativa por linha — apenas operações entre colunas da mesma linha (ex: `[Quantidade] * [PrecoUnitario]`).
- **Por que importa**: coluna calculada com cálculo trivial é materializada em memória pra cada linha. Em tabela-fato de 4M linhas isso infla o `.pbix` em ~80MB e não traz benefício — o mesmo cálculo pode ser medida sem custo.
- **Como corrigir**: deletar coluna, criar medida equivalente.
- **Snippet**:
```dax
Valor Total Vendas =
SUMX(
    Vendas,
    Vendas[Quantidade] * Vendas[PrecoUnitario]
)
```

### `perf-distinctcount-high-cardinality`
- **Severidade**: `critical`
- **Como detectar**: medida usando `DISTINCTCOUNT()` em coluna do tipo string que esteja em tabela com > 100k linhas.
- **Por que importa**: `DISTINCTCOUNT` em string com alta cardinalidade força scan + dedupe a cada contexto de filtro. Tempo de query escala mal e mata visuais com slicer dinâmico.
- **Como corrigir**: criar coluna `Key` (inteiro sequencial) na tabela origem, usar `DISTINCTCOUNT` na key inteira. Inteiro é ~10x mais rápido que string em scan + dedupe.

### `perf-string-as-key`
- **Severidade**: `medium`
- **Como detectar**: coluna usada como chave de relacionamento (`isKey: true` ou referenciada em `relationship`) com `dataType: string` e cardinalidade > 50k.
- **Por que importa**: relacionamentos por string são significativamente mais lentos que por inteiro. Engine precisa hash da string em todo join.
- **Como corrigir**: criar coluna inteira sequencial, usar como chave nova. Manter string como atributo.

### `perf-unused-columns`
- **Severidade**: `light`
- **Como detectar**: coluna em tabela-fato que não aparece em nenhuma medida, nenhum relacionamento e não é referenciada em visual report (verificar `.Report/`).
- **Por que importa**: coluna não usada ocupa memória sem retorno. Em tabela grande, soma rápido.
- **Como corrigir**: marcar `isHidden: true` (mínimo) ou remover (ideal).

### `perf-auto-date-time`
- **Severidade**: `medium`
- **Como detectar**: configuração `autoDateTime: true` ou tabelas escondidas começando com `LocalDateTable_` ou `DateTableTemplate_`.
- **Por que importa**: Auto Date/Time cria uma tabela calendário oculta pra cada coluna data — modelo médio acumula dezenas dessas, infla `.pbix`, polui medidas com prefixo `[Date].[Date]`.
- **Como corrigir**: desligar Auto Date/Time globalmente (Options → Data Load → Time intelligence). Usar tabela calendário dedicada.

---

## Família 4 · NAMING

### `naming-mixed-conventions`
- **Severidade**: `medium`
- **Como detectar**: detectar 3+ padrões diferentes em medidas/colunas: `camelCase`, `snake_case`, `Title Case`, `MAIÚSCULAS`, `kebab-case`.
- **Por que importa**: modelo mistura `Total Vendas`, `tot_vendas`, `SalesTotal` e `VENDAS_TOTAL`. Quem herdar gasta 2x do tempo procurando porque nome não é previsível. Em time grande, vira fonte de duplicação (3 medidas fazendo o mesmo).
- **Como corrigir**: padronizar pra **Title Case PT-BR com prefixo de tipo** (recomendação SQLBI). Exemplos: `Total Vendas`, `% Margem Bruta`, `# Pedidos`, `Δ MoM Vendas`. Padroniza nas medidas afetadas e bloqueia em PR review.

### `naming-no-table-prefix`
- **Severidade**: `light`
- **Como detectar**: tabelas dimensão sem prefixo `dim ` / `dim_` / `Dim` E tabelas fato sem prefixo `fact ` / `fact_` / `Fact`.
- **Por que importa**: prefixo facilita identificação imediata do papel da tabela no modelo. Sem prefixo, alguém abre o painel de tabelas e tem que ler o conteúdo pra entender.
- **Como corrigir**: padronizar com `dim ` (espaço) ou `dim_` em dimensões, `fact ` ou `fact_` em fatos. Manter convenção em todo o modelo.

### `naming-acronym-no-context`
- **Severidade**: `light`
- **Como detectar**: medidas/colunas com nomes ≤ 4 caracteres ou só acrônimos sem palavra em PT (ex: `KPI1`, `MRR`, `LTV`, `CAC`).
- **Por que importa**: acrônimo sem contexto vira jargão tribal — só quem criou entende. Próximo analista precisa adivinhar.
- **Como corrigir**: expandir acrônimo OU adicionar descrição na medida (`measure 'MRR' description: "Monthly Recurring Revenue — soma de assinaturas ativas no mês"`).

---

## Família 5 · DAX

### `dax-divide-operator`
- **Severidade**: `medium`
- **Como detectar**: medidas usando operador `/` entre dois números/medidas, sem `DIVIDE()`.
- **Por que importa**: operador `/` em DAX gera **Infinity** ou **NaN** quando denominador é 0 — visual mostra "∞" ou erro vermelho. Em dashboard de diretoria isso vira ligação às 7h da manhã.
- **Como corrigir**: substituir `X / Y` por `DIVIDE(X, Y, BLANK())`. DIVIDE retorna BLANK se denominador é 0 — visual fica vazio (esperado), não quebra.

### `dax-unused-variables`
- **Severidade**: `light`
- **Como detectar**: medida com `VAR` declarada que não é usada no `RETURN` nem em outras VARs.
- **Por que importa**: variáveis não usadas adicionam ruído visual e indicam refatoração incompleta. Quem ler vai perder tempo entendendo se faz parte da lógica.
- **Como corrigir**: remover variáveis órfãs.

### `dax-nested-calculate`
- **Severidade**: `medium`
- **Como detectar**: medidas com `CALCULATE` aninhado em outro `CALCULATE` (regex `CALCULATE\s*\([^)]*CALCULATE`).
- **Por que importa**: CALCULATE aninhado é quase sempre evitável e dificulta debug do contexto de filtro. Geralmente indica que a medida precisa ser refatorada com VARs.
- **Como corrigir**: extrair o `CALCULATE` interno em `VAR`, usar a variável no `CALCULATE` externo. Lógica fica linear, debug fica trivial.

### `dax-iterators-on-large-tables`
- **Severidade**: `medium`
- **Como detectar**: `SUMX/AVERAGEX/RANKX/FILTER` direto sobre tabela-fato com > 1M linhas, sem `CALCULATETABLE` ou filtro contextual reduzindo escopo.
- **Por que importa**: iteradores em tabela grande sem pré-filtro escaneiam linha a linha. Em fato de 5M linhas, query trava 5–15s.
- **Como corrigir**: reduzir escopo com `CALCULATETABLE` antes de iterar, ou usar agregações (`SUM`, `COUNT`) quando possível.

---

## Família 6 · DOCUMENTAÇÃO

### `doc-measures-no-description`
- **Severidade**: `light`
- **Como detectar**: medidas sem propriedade `description` no tmdl.
- **Por que importa**: medida sem descrição é caixa-preta pra quem usa. Usuário de relatório vê o nome e tenta adivinhar o que significa.
- **Como corrigir**: adicionar `description:` em toda medida com 1-2 frases explicando o que calcula e em qual unidade.
- **Snippet**:
```tmdl
measure 'Total Vendas Líquidas'
    description: "Receita total descontando devoluções e cancelamentos. Em R$, sem impostos."
    formatString: "#,0.00"
```

### `doc-tables-no-description`
- **Severidade**: `light`
- **Como detectar**: tabelas sem propriedade `description`.
- **Por que importa**: tabela sem descrição obriga novo analista a abrir e investigar pra entender o papel.
- **Como corrigir**: adicionar descrição curta indicando origem dos dados e granularidade.

### `doc-no-business-key-marked`
- **Severidade**: `medium`
- **Como detectar**: tabelas dimensão sem coluna marcada como `isKey: true`.
- **Por que importa**: sem chave de negócio explícita, Power BI escolhe coluna automaticamente — geralmente errado, gera relacionamentos suspeitos.
- **Como corrigir**: marcar explicitamente a chave de negócio em cada dimensão.

### `data-source-personal-path`
- **Severidade**: `medium`
- **Como detectar**: parsing de `expressions.tmdl` ou expressões M dentro de partições — procurar paths que contenham `C:\Users\<nome>\`, `G:\Drives compartilhados\`, `\Users\Leo\`, `OneDrive\`, ou outros indicadores de path pessoal/local hardcoded.
- **Por que importa**: caminho hardcoded de Google Drive, OneDrive ou pasta de usuário específico **quebra refresh em qualquer máquina que não seja a do criador**. Pra modelo que vai pra aluno (curso) ou pra time (handoff), isso é incidente de suporte garantido — pessoa baixa o projeto, abre, refresh quebra, suporte explode.
- **Como corrigir**:
  - Transformar em **parâmetro do modelo** com default vazio + instrução ao abrir
  - OU usar **OneDrive shared link** que funcione em qualquer login autorizado
  - OU compartilhar dados como **dataset publicado** no Power BI Service em vez de Excel local
  - OU usar **Dataflow** centralizado
- **Snippet** (parametrização):
```tmdl
expression CaminhoBase = "" meta [
    IsParameterQuery=true,
    Type="Text",
    IsParameterQueryRequired=true
]
    annotation PBI_ResultType = Text
```

---

## Como aplicar (algoritmo geral)

Pra cada `.tmdl` na pasta `SemanticModel/`:

1. Parse o arquivo extraindo: tabelas, colunas, medidas, relacionamentos, propriedades
2. Pra cada check listado acima:
   - Aplicar a heurística de detecção
   - Se positivo → registrar issue com todos os campos preenchidos
3. Após varrer tudo, ordenar issues por severidade (crítico → médio → leve), depois por categoria (concentração)
4. Calcular score conforme fórmula no SKILL.md

## Tom dos textos no relatório

Os textos de "por que importa" e "como corrigir" acima são **base canônica** mas devem ser **adaptados ao contexto real** do projeto auditado:

- Substituir números genéricos pelos reais ("4M linhas em Vendas" se a tabela tem mesmo 4M)
- Mencionar nomes reais das tabelas/colunas/medidas afetadas
- Manter tom Xperiun (provocativo, direto, com metáforas concretas)

Exemplo de adaptação:
- **Genérico**: "Coluna calculada com cálculo trivial é materializada em memória"
- **Adaptado**: "Vendas[ValorTotal] (Quantidade × Preço) é materializada nas suas 4.2M linhas — infla o pbix em ~80MB sem ganho real"
