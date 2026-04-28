# Claude Skills · Power BI

> **3 skills oficiais do Claude pra documentar, auditar e criar DAX no Power BI. Funciona no Claude.ai grátis. Zero programação. 2 minutos pra instalar.**

[![Xperiun](https://img.shields.io/badge/by-Xperiun-7099FF?style=flat-square)](https://pages.xperiun.com)
[![Free](https://img.shields.io/badge/Claude-Free%20OK-3FD1B8?style=flat-square)](https://claude.ai)
[![License](https://img.shields.io/badge/license-MIT-E8C9A0?style=flat-square)](LICENSE)

---

## O problema

Você herda um `.pbix` de 400 medidas sem documentação. Gasta 3 semanas só pra entender o modelo. Aí precisa criar uma medida e digita DAX no editor pequenininho do Power BI Desktop. E quando termina, ninguém sabe se o modelo tá bem feito.

**3 tarefas chatas, repetitivas, que rodam toda semana:**

1. **Documentar** o modelo
2. **Auditar** se tá bem feito
3. **Criar** medidas DAX

Esse repo automatiza as 3 — e roda **direto no Claude.ai grátis**.

---

## As 3 skills

### 🩺 `pbi-modelo-review`

**Audita o modelo Power BI e devolve scorecard 0-100 + lista priorizada de issues.**

- 23 checks em 6 famílias (modelagem, relacionamentos, performance, naming, DAX, documentação)
- Output: relatório HTML visual (estilo Lighthouse) + markdown
- Detecta: bi-direcional desnecessário, Auto Date/Time, FAT FACT, DAX antipatterns, paths pessoais hardcoded
- Roda em ~3-5 min em modelo médio

### 📚 `pbi-doc`

**Gera mini-site de documentação navegável + 5 markdowns Git-friendly.**

- Sidebar fixa com navegação · busca client-side · scroll spy
- Documenta: tabelas + colunas tipadas + medidas DAX explicadas em PT + relacionamentos (SVG visual) + dependências
- Cada medida vira card expansível com DAX + "O que faz" + "Como funciona" + "É usada por"
- Output: 1 HTML standalone + 5 .md (Git diff entre versões mostra evolução do modelo)

### ⚡ `pbi-dax-create`

**Cria medida DAX a partir de descrição em PT, respeitando o modelo existente.**

- Lê tabelas/colunas/medidas reais do modelo
- Detecta padrão de naming (Title Case, prefixos, sufixos)
- Aplica boas práticas DAX automaticamente (DIVIDE em vez de /, SUMX vs SUM, time intelligence)
- Avisa se medida similar já existe (não duplica)
- Pergunta sempre antes de aplicar (modo seguro)

---

## Downloads

| Skill | `.zip` |
|---|---|
| 🩺 Auditar modelo | [`pbi-modelo-review.zip`](claude-web/pbi-modelo-review.zip) |
| 📚 Documentar modelo | [`pbi-doc.zip`](claude-web/pbi-doc.zip) |
| ⚡ Criar medida DAX | [`pbi-dax-create.zip`](claude-web/pbi-dax-create.zip) |

**Setup 2 minutos · 3 caminhos (Web · Desktop · Code):** ver [INSTALL.md](INSTALL.md).

**Pré-requisito:** seu projeto Power BI salvo como **PBIP** (não `.pbix`). [Como converter em 30s](INSTALL.md#antes-de-começar--pré-requisitos).

---

## Por que esse caminho (e não MCP)

A maioria dos tutoriais de IA + Power BI ensina via **MCP server** — server rodando ao vivo, dados indo pra API, "Sempre permitir" clicado no susto. **Em cliente corporativo isso não passa.**

Esse repo segue **PBIP + Claude Skills** (oficial Anthropic):

| MCP | Claude Skills (este repo) |
|---|---|
| Server rodando ao vivo | Anexa só os arquivos .tmdl que você quer |
| "Sempre permitir" clicado | Você controla o que sobe a cada conversa |
| Dados indo pra API sem filtro | Você escolhe arquivo a arquivo |
| Proibido em cliente corporativo | LGPD-compatível · auditável |
| Precisa Power BI Premium / Pro+ | Claude Free funciona |

---

## Onde rodar — 3 ambientes Claude

Cada skill **detecta automaticamente** onde tá rodando e adapta. Escolhe um:

| Ambiente | Pra quem | Plano | Anexar arquivos? | Output |
|---|---|---|---|---|
| **Claude.ai (web)** ⭐ | Default · você sem nada instalado · começar agora | Free serve | A cada conversa (drag-drop) | Artifact HTML inline + download |
| **Claude Desktop** | Já usa o app nativo Mac/Windows | Free serve · Pro recomendado | A cada conversa (drag-drop) | Igual Web · janela dedicada · atalho de teclado |
| **Claude Code** | Dev/analista com Git · workflow contínuo · uso intenso | Pro+ na prática | **Lê sozinha** do disco | Salva direto em `_review/`, `_docs/` no projeto · edita `.tmdl` (com aprovação) |

⭐ = recomendado se você não tem certeza.

**Como instalar em cada um · passo a passo completo:** [INSTALL.md](INSTALL.md).

---

## Como usar — exemplos práticos

### Documentar um modelo

```
[claude.ai · você]
> Quero documentar meu modelo Power BI

[claude · skill ativa]
> Anexe os .tmdl da pasta SemanticModel/ (ou suba zipado)

[você anexa]
> [ZIP com 6 .tmdl]

[claude · 5 min depois]
> [Artifact com mini-site HTML completo + 5 markdowns]
```

### Auditar um modelo

```
> audita esse modelo
> [anexa .tmdl]
> [recebe scorecard 0-100 + 23 issues priorizados]
```

### Criar uma medida DAX

```
> cria uma medida pra ticket médio
> [se preciso, skill pede pra anexar Medidas.tmdl pra entender naming]
> [recebe 3 sugestões de nome + DAX completo + explicação + onde colar]
```

---

## Arquitetura

Cada skill segue o padrão **Anthropic Skills**:

```
pbi-modelo-review/
├── SKILL.md              # frontmatter + manual completo
├── references/
│   ├── checks.md         # 23 checks técnicos detalhados
│   └── design-tokens.md  # tokens visuais do output HTML
└── templates/
    ├── relatorio.html    # template HTML parametrizado
    └── relatorio.md      # template markdown
```

Cada skill é **self-contained** — não depende das outras. Pode instalar só uma se quiser.

Pra usuários do **Claude Code**, o repo também tem `claude-code/` com as 3 skills configuradas pra uso direto:

```bash
# Modo global (todos os projetos)
cp -r claude-code/* ~/.claude/skills/

# Modo por projeto
cp -r claude-code/* /seu-projeto-pbip/.claude/skills/
```

---

## Roadmap

**v0.1 (atual)** — 3 skills core: review, doc, create.

**v0.2 (planejado)**:
- `pbi-dax-refactor` — refatora medidas existentes pro padrão SQLBI
- `pbi-perf` — auditoria focada em performance
- `pbi-from-csv` — gera modelo dimensional inicial a partir de CSV/Excel

**v1.0 (longo prazo)**:
- Plugin VS Code com snippets
- Suporte a `.bim` (TMSL) além de PBIP
- GitHub Action que roda audit em PR automaticamente

---

## Contribuindo

Issues e PRs bem-vindos. Sugestões de novas skills via [GitHub Discussions](https://github.com/xperiun/claude-code-powerbi-skills/discussions).

---

## Sobre a Xperiun

[Xperiun](https://xperiun.com) é a escola de Power BI + Dados + IA pra analistas e líderes brasileiros que querem virar **incomparáveis**.

Esse repo é parte do nosso compromisso de manter **conhecimento de ponta acessível e gratuito** — quem quiser ir mais fundo (curso completo, mentoria, comunidade), conhece o programa em [xperiun.com](https://xperiun.com).

**Siga**: [@leokarpa](https://instagram.com/leokarpa) · [@xperiun](https://instagram.com/xperiun)

**Landing oficial**: [pages.xperiun.com/skills-power-bi](https://pages.xperiun.com/skills-power-bi)

---

## Licença

MIT. Use, modifique, distribua. Se quiser, dá uma estrela ⭐ no repo — é o sinal que mais ajuda a gente saber que tá sendo útil.

---

*Construído com 🩵 pela equipe Xperiun. v0.1 · 2026-04*
