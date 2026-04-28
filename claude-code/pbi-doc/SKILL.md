---
name: pbi-doc
description: Documenta projeto Power BI (PBIP) inteiro em markdown estruturado + HTML navegável (mini-site de doc). Use quando o usuário pedir "documenta esse projeto", "gera doc do power bi", "explica esse modelo", "preciso entregar handoff", ou apontar uma pasta PBIP pra mapeamento descritivo (não auditoria).
---

# /pbi-doc — Documentação automática de Power BI

Gera documentação completa de um projeto Power BI (formato PBIP) em duas formas:
- **Markdown** versionável Git (5 arquivos: overview, tabelas, medidas, relacionamentos, dependências)
- **HTML standalone** navegável (mini-site com sidebar fixa, busca, syntax highlight em DAX)

A doc descreve o que **existe** no modelo — tabelas, colunas tipadas, medidas com DAX explicadas em PT, relacionamentos com cardinalidade, grafo de dependências entre medidas. **Não opina sobre qualidade** (essa é função da `/pbi-modelo-review`).

## Quando usar

- Analista herdou um `.pbix` de N tabelas e M medidas e precisa entender rápido
- Líder pedindo handoff documentado pra outro time
- Pré-onboarding de novo membro no time de dados
- Precisa de "manual de uso" do modelo pra circular junto com o relatório
- Documentação contínua: rodar a cada release pra manter doc viva no Git

**Não usar quando:**
- Quer auditoria de qualidade / anti-patterns → use `/pbi-modelo-review`
- Quer criar uma medida nova → use `/pbi-dax-create`
- Quer só extrair lista de medidas em CSV (skill futura `/pbi-export-medidas`)

## Pré-requisitos

1. **Projeto em formato PBIP** (Power BI Project) — pasta com `.SemanticModel/` e `.Report/`. Se o usuário só tem `.pbix`, instruir conversão **antes**:
   - `Power BI Desktop → File → Save as → Power BI Project (.pbip)`
2. Acesso aos arquivos `.tmdl` (via filesystem ou upload — ver "Modos de execução" abaixo)

Se faltar PBIP, retornar mensagem curta:
> Esse projeto ainda está em `.pbix` (binário). Pra eu documentar, salva como Power BI Project: `File → Save as → Power BI Project (.pbip)`. Vira uma pasta de texto e aí eu consigo ler. Avisa quando converter.

E encerrar — não tentar nada.

## Modos de execução

A skill detecta automaticamente o ambiente e adapta input/output:

### Modo Code (Claude Code · Desktop · file-based)
- **Detecção**: tenho acesso a filesystem e a pasta atual contém `.SemanticModel/`
- **Input**: leio automaticamente os `.tmdl` de `./SemanticModel/`
- **Output**: salvo em `./_docs/index.html` + 5 markdowns (`00-overview.md` a `04-dependencias.md`) na raiz do projeto Power BI
- **Idempotente**: rodar 2x sobrescreve

### Modo Web (Claude.ai · upload-based)
- **Detecção**: não tenho acesso a filesystem (claude.ai web)
- **Input**: peço ao usuário pra anexar os arquivos:
  > Pra eu documentar, anexe nesse chat:
  > - Os arquivos `.tmdl` da pasta `SemanticModel/definition/` (model.tmdl, relationships.tmdl, expressions.tmdl se houver)
  > - Os arquivos da pasta `SemanticModel/definition/tables/` (1 .tmdl por tabela, **excluindo** as auto-date `LocalDateTable_*` e `DateTableTemplate_*`)
  >
  > Pode arrastar individualmente ou zipar a pasta `SemanticModel/` e subir 1 ZIP.
- **Output**:
  - HTML completo (mini-site navegável) como **artifact** (Claude.ai renderiza inline + botão de download)
  - Os 5 markdowns como blocos de código no chat (copiáveis um a um) OU 1 ZIP com todos
- **Não persiste**: cada conversa nova requer novo upload

### Detecção automática

Verificar se a pasta `.SemanticModel/` é acessível via filesystem:
- ✅ Sim → Modo Code (file-based)
- ❌ Não → Modo Web (peço uploads)

Se ambíguo, perguntar uma vez:
> Você tá rodando isso no **Claude Code** (CLI/IDE com acesso à pasta) ou no **claude.ai** (web)? Pra Code eu leio a pasta sozinho; pra web preciso que você suba os arquivos.

### Trade-offs por modo

| Aspecto | Code | Web |
|---|---|---|
| Setup | 1× (instala skill) | 0 (só sobe arquivo) |
| Por uso | comando único | anexar TMDL toda vez |
| Modelo grande (>200 medidas) | OK | pode estourar contexto Free |
| Persistência | salva em disco | só na conversa (baixar artifact) |
| Custo | tokens Claude Code | tokens claude.ai (Free incluído) |

## Inputs

- **Escopo** (opcional, default: `tudo`)
  - `tudo` → todos os 5 arquivos
  - `só medidas` → só `02-medidas.md` (útil pra checar mudanças após refator)
  - `só tabelas` → só `01-tabelas.md`
  - `só relacionamentos` → só `03-relacionamentos.md`
  - `tabela X` → restringe descrição às tabelas específicas (separadas por vírgula)

Se não especificado, perguntar **uma vez**:
> Documento o projeto inteiro (5 arquivos) ou prefere algo específico — só medidas, só relacionamentos, ou tabelas específicas?

## Processo

### 1. Detectar e mapear

- Confirmar `.SemanticModel/`
- Listar `.tmdl` em `./SemanticModel/tables/` (excluir `LocalDateTable_*` e `DateTableTemplate_*` — são auto-geradas, não fazem parte da doc)
- Ler `model.tmdl`, `relationships.tmdl`, `expressions.tmdl` (se existir)
- Ler **todos** os `.tmdl` de tabelas
- Inventariar:
  - **Tabelas**: nome, tipo (fato/dim/measures-only), descrição, granularidade inferida, lista de colunas, partição/source M
  - **Medidas**: nome, expressão DAX, displayFolder (agrupa), formatString, descrição (se existir), referências a outras medidas
  - **Relacionamentos**: from, to, cardinalidade, direção, ativo
  - **Dependências**: medida X usa medida Y; medida Z usa coluna W

### 2. Gerar 5 arquivos markdown

Ler templates em `templates/` e preencher com dados reais. Salvar em `./_docs/` na raiz do projeto Power BI:

| Arquivo | Conteúdo |
|---|---|
| `_docs/00-overview.md` | Sumário (N tabelas, N medidas, N relacionamentos, fontes, propósito inferido) |
| `_docs/01-tabelas.md` | Cada tabela: descrição, granularidade, colunas tipadas, source M (resumo) |
| `_docs/02-medidas.md` | Agrupadas por displayFolder. Cada uma: nome, DAX, explicação PT linha-a-linha |
| `_docs/03-relacionamentos.md` | Lista detalhada + diagrama em ASCII art (matriz simples) |
| `_docs/04-dependencias.md` | Grafo: árvore "medida X → usa Y → usa Z" + lista reverse "Y é usada por: A, B, C" |

### 3. Gerar HTML standalone

**🚨 REGRA INVIOLÁVEL — usar templates/relatorio.html LITERAL:**

1. **LER** `templates/relatorio.html` — esse arquivo já tem **todo o CSS, todo o HTML estrutural, todos os tokens DS v4 (Bebas Neue, accent-gold, gold-grid + beams animados, orb-v2 elipses blue/purple, riscas section+section::before, brackets), todo o JS de scroll spy/busca**. CSS são ~600 linhas inline + HTML completo com gold-grid, sidebar, topbar, sections.

2. **SUBSTITUIR APENAS os placeholders `{{...}}`** pelos valores reais derivados dos `.tmdl`. Lista completa dos placeholders está em `references/escopo.md` desta skill (seção "Placeholders do `templates/relatorio.html`"). Todos os blocos `{{...}}_HTML` são gerados pelo Claude com base no inventário do modelo.

3. **PROIBIDO:**
   - ❌ Trocar o CSS por outro
   - ❌ Inventar nova paleta de cores (usar SÓ os tokens do template: `--accent-gold-bright #E8C9A0`, `--accent-glow #7099FF`, `--neon-magenta #C47FFF`, etc.)
   - ❌ Mudar fontes (DS v4 usa Bebas Neue + Barlow Condensed + Outfit + JetBrains Mono — nada de Segoe UI, Arial, system-ui)
   - ❌ Remover o `<div class="gold-grid">`, os `<div class="section-orb">`, ou qualquer ornamento decorativo do template
   - ❌ Gerar HTML "do zero" porque parece mais fácil — **isso queima toda a identidade visual Xperiun**
   - ❌ **Tocar em qualquer coisa dentro de comentários `<!-- ... -->`** — comentários são instruções pra você, não conteúdo a substituir. Mantém como tá.
   - ❌ **Tocar em `<style>...</style>` ou `<script>...</script>`** — CSS e JS ficam intocados.

4. **🚨 ENCODING — UTF-8 PURO, sem escape.** Caracteres PT-BR (`ã`, `ç`, `é`, `á`, `õ`, `ê`, `í`, `ú`) e símbolos especiais (`├`, `└`, `─`, `→`, `↔`, `↑`, `↓`, `⚠`, `·`, `—`) devem aparecer como **caracteres reais UTF-8**, NÃO como sequências escapadas/HTML entities/mojibake.
   - ✅ Correto: `dependências`, `└─`, `→`, `Incomparáveis`
   - ❌ Errado (mojibake): `dependÃªncias`, `âââ`, `â`, `IncomparÃ¡veis`
   - ❌ Errado (entities desnecessárias): `depend&ecirc;ncias`
   - **Sintoma de erro:** se algum acento aparece como sequência de 2-3 chars estranhos (`Ã£`, `â`, `Ã©`), o parser HTML pode quebrar e o resto da página renderiza como texto cru. Refaz garantindo UTF-8.

5. **SALVAR** em `./_docs/index.html` (modo Code) ou retornar como artifact (modo Web).

6. **Como deve parecer:** fundo `#0D0C0E` quase preto · gold-grid de papel pautado dourado animado caindo · orbs azul/roxo em cada seção · risca dourada entre seções · cards `var(--gradient-surface)` com border `--border-faint` · números em Bebas Neue gold · DAX com syntax highlight via spans `.k .f .s .c`. Estilo "editorial premium dark" — não dashboard genérico tipo Vercel/Stripe.

7. **Sintomas de erro:**
   - Cores como `#f5a623` (laranja) ou `#7c6af7` (roxo genérico), ou fonte `'Segoe UI'` → ignorou o template, refaz.
   - Acentos como `Ã£` ou `â` → encoding quebrado, refaz com UTF-8 puro.
   - Texto solto sem quebras (SVG/tabela aparecendo como prosa) → encoding mojibake quebrou o parser HTML, refaz.

### 4. Resumir no chat

Mensagem curta:
- Quantidade do que foi documentado (5 tabelas, 19 medidas, 4 relacionamentos)
- Path dos arquivos gerados
- Sugestão: "Abre `_docs/index.html` pra ver navegável"

## Outputs

```
[raiz do projeto Power BI do usuário]/
├── SemanticModel/                  ← input (não tocar)
├── Report/                         ← input (não tocar)
└── _docs/                          ← OUTPUT da skill
    ├── 00-overview.md
    ├── 01-tabelas.md
    ├── 02-medidas.md
    ├── 03-relacionamentos.md
    ├── 04-dependencias.md
    └── index.html                  ← versão visual standalone
```

## Edge cases

| Cenário | O que fazer |
|---|---|
| Sem `.SemanticModel/` | Mensagem de pré-requisito (PBIP), encerra |
| Pasta `_docs/` já existe | **Sobrescrever** (idempotente) — avisar no chat |
| Modelo gigante (>200 medidas) | Avisar tempo + processar em chunks |
| Tabelas auto-date (`LocalDateTable_*`, `DateTableTemplate_*`) | **Excluir da doc** — são tabelas-fantasma, não fazem parte do modelo intencional |
| Medida com DAX muito complexo (>30 linhas) | Mostrar DAX completo + explicar em **blocos** (se / agg / contexto) |
| Modelo sem nenhuma descrição declarada | Inferir propósito a partir de naming + estrutura, mas sinalizar "descrição inferida (não há `description:` declarado)" |

## Tom da documentação

Estilo Xperiun:
- **PT-BR direto, não robótico**. Em vez de "A tabela X possui Y colunas", escrever "Vendas — fato principal do modelo, 1 linha = 1 item de NFe, 12 colunas (5 chaves + 7 atributos)"
- Pode usar metáforas concretas pra explicar DAX complexo
- Manter tom de "colega sênior explicando o modelo pro novo membro do time"
- PT-BR com **todos os acentos** (regra inviolável CLAUDE.md)

Exemplos de **bom** vs **ruim**:

❌ Ruim: "A medida 'Faturamento' calcula o resultado da multiplicação entre QtdItens e PrecoUnitario."

✅ Bom: "**Faturamento** — multiplica quantidade × preço linha-a-linha em fVendas e soma o total. É a medida-mãe: várias outras (Margem Bruta, %YoY, etc) dependem dela."

## Idempotência e segurança

- Rodar 2x **sobrescreve** `_docs/`
- Não modifica nada em `.SemanticModel/` ou `.Report/` — somente leitura
- Não commita nada (segue regra git inviolável CLAUDE.md)
- Operação 100% local — zero rede, zero XMLA

## Branding

HTML tem footer fixo:
- "Doc gerada por Claude Code + /pbi-doc · Xperiun"
- CTA: "Quero usar esse skill no meu Power BI →"
- Meta: "XPERIUN · O Sistema Operacional dos Incomparáveis · pages.xperiun.com"

Branding sempre Xperiun.

## Tempo típico

- Modelo pequeno (≤30 tabelas, ≤80 medidas): **2–4 min**
- Modelo médio (~50 tabelas, ~150 medidas): **5–8 min**
- Modelo grande (>200 medidas): **10–15 min**

Avisar se >5min esperados.

## Versão atual

`v0.1` — protótipo interno Xperiun OS. Quando estabilizar, vira pacote no repo público `xperiun/claude-code-powerbi-skills` (lead magnet).
