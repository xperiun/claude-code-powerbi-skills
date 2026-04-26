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

## Instalação rápida (Claude.ai · Free funciona)

> **Tempo total**: 2 minutos por skill.

### 1. Baixa o `.zip` da skill que você quer usar

| Skill | Download |
|---|---|
| 🩺 Auditar modelo | [`pbi-modelo-review.zip`](releases/pbi-modelo-review.zip) |
| 📚 Documentar modelo | [`pbi-doc.zip`](releases/pbi-doc.zip) |
| ⚡ Criar medida DAX | [`pbi-dax-create.zip`](releases/pbi-dax-create.zip) |

### 2. Sobe no claude.ai

- Abre **claude.ai** (qualquer plano · Free funciona)
- Vai em **Personalizar → Habilidades** (sidebar esquerda)
- Clica **+ Criar habilidade → Fazer upload de uma habilidade**
- Arrasta o `.zip` que você baixou

Pronto. A skill aparece em "Habilidades pessoais" e fica disponível em qualquer conversa.

### 3. Usa

Abre uma conversa nova e escreve, por exemplo:

```
audita esse modelo aqui
```

ou

```
documenta esse modelo
```

ou

```
cria uma medida pra calcular o ticket médio
```

A skill vai pedir os arquivos `.tmdl` do seu projeto Power BI. Você anexa (arrastando ou via botão de anexo) e a skill processa.

---

## Pré-requisitos do seu lado

- **Conta Claude** (Free serve · Pro tem limites maiores)
- **Power BI Desktop** com seu projeto salvo como **PBIP**

### Como salvar como PBIP (1 vez só)

1. Abre seu `.pbix` no Power BI Desktop
2. `File → Save as → Power BI Project (.pbip)`
3. Vira uma **pasta** com `SemanticModel/` dentro

A skill lê os `.tmdl` dessa pasta `SemanticModel/`. Tudo texto, tudo seguro, tudo local — nada vai pra rede além do que você anexa no Claude.

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

## Modos de uso

Cada skill **detecta automaticamente** onde tá rodando e adapta:

### Modo Web (Claude.ai · Desktop · grátis)
- Você anexa os `.tmdl` no chat (individual ou ZIP)
- Skill devolve **HTML como artifact** (renderiza inline + botão de download) e **markdown como bloco copiável**
- Não persiste — cada conversa nova requer novo upload
- **Funciona com Claude Free**

### Modo Code (Claude Code CLI/IDE)
- Skill lê automaticamente a pasta `.SemanticModel/` do diretório atual
- Salva output em `_review/`, `_docs/` na raiz do projeto Power BI
- Edita `.tmdl` direto se você autorizar
- Precisa **Claude Pro ou superior** (Free estoura limite rápido)
- Recomendado pra uso intenso e workflow Git

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

Pra usuários do **Claude Code**, o repo também tem `.claude/skills/` com as 3 skills configuradas pra uso direto:

```bash
# Modo global (todos os projetos)
cp -r .claude/skills/* ~/.claude/skills/

# Modo por projeto
cp -r .claude/skills/* /seu-projeto-pbip/.claude/skills/
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
