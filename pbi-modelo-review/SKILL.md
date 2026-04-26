---
name: pbi-modelo-review
description: Audita modelo Power BI (PBIP) e gera relatório priorizado com anti-patterns, score 0-100 e recomendações acionáveis. Use quando o usuário pedir "audita esse modelo", "revisa esse power bi", "tá bem feito?", "manda ver no review", ou apontar uma pasta PBIP pra análise crítica.
---

# /pbi-modelo-review — Auditoria de modelo Power BI

Audita um projeto Power BI (formato PBIP) e devolve relatório priorizado com:
- Score geral (0–100)
- Issues agrupados por severidade (crítico / médio / leve)
- Distribuição por categoria (6 famílias de check)
- Cada issue com: o que é, onde está, por que importa, como corrigir (snippet quando aplicável)

## Quando usar

- Analista herdou um `.pbix` e quer mapa rápido de qualidade
- Líder de dados pedindo auditoria antes de escalar/migrar modelo
- Consultor entrando em cliente novo
- Pré-handoff entre times
- Você quer um "linter" rodando antes de subir pra produção

**Não usar quando:**
- Quer apenas documentação descritiva → use `/pbi-doc`
- Quer criar uma medida nova → use `/pbi-dax-create`
- Quer otimização de performance específica de uma query (skill futura `/pbi-perf`)

## Pré-requisitos

1. **Projeto em formato PBIP** (Power BI Project) — pasta com `.SemanticModel/` e `.Report/`. Se o usuário só tem `.pbix`, instruir conversão **antes** de prosseguir:
   - `Power BI Desktop → File → Save as → Power BI Project (.pbip)`
2. Acesso aos arquivos `.tmdl` (via filesystem ou upload — ver "Modos de execução" abaixo)

Se faltar PBIP, retornar mensagem curta:
> Esse projeto ainda está em `.pbix` (binário). Pra eu auditar, salva como Power BI Project: `File → Save as → Power BI Project (.pbip)`. Vira uma pasta de texto e aí eu consigo ler. Avisa quando converter.

E encerrar — não tentar rodar nada.

## Modos de execução

A skill detecta automaticamente o ambiente e adapta input/output:

### Modo Code (Claude Code · Desktop · file-based)
- **Detecção**: tenho acesso a filesystem do usuário e a pasta atual contém `.SemanticModel/`
- **Input**: leio automaticamente os `.tmdl` da pasta `./SemanticModel/`
- **Output**: salvo em `./_review/index.html` + `./_review/relatorio.md` na raiz do projeto Power BI
- **Idempotente**: rodar 2x sobrescreve

### Modo Web (Claude.ai · upload-based)
- **Detecção**: não tenho acesso a filesystem (claude.ai web)
- **Input**: peço ao usuário pra anexar os arquivos:
  > Pra eu auditar, anexe nesse chat:
  > - Os arquivos `.tmdl` da pasta `SemanticModel/definition/` (model.tmdl, relationships.tmdl, expressions.tmdl)
  > - Os arquivos da pasta `SemanticModel/definition/tables/` (1 .tmdl por tabela)
  >
  > Pode arrastar todos de uma vez ou zipar a pasta `SemanticModel/` e subir o ZIP.
- **Output**:
  - HTML completo como **artifact** (Claude.ai renderiza inline com botão de download)
  - Markdown como bloco de código copiável no chat
- **Não persiste**: cada conversa nova requer novo upload

### Detecção automática

Verificar se a pasta `.SemanticModel/` é acessível via filesystem:
- ✅ Sim → Modo Code (file-based)
- ❌ Não → Modo Web (peço uploads)

Se ambíguo, perguntar uma vez:
> Você tá rodando isso no **Claude Code** (CLI/IDE com acesso à pasta) ou no **claude.ai** (web)? Pra Code eu leio a pasta sozinho; pra web preciso que você suba os arquivos.

## Inputs

- **Severidade mínima** (opcional, default: `tudo`)
  - `tudo` → reporta crítico + médio + leve
  - `crítico+médio` → ignora leves (foco no que cobra juros)
  - `só crítico` → só o que bloqueia produção

Se o usuário não especificar, perguntar **uma vez** antes de rodar:
> Quer auditoria completa (todos os níveis) ou só os críticos+médios? Default é completa.

## Processo

### 1. Detectar e mapear o projeto

- Confirmar existência de `./SemanticModel/` (ou `*.SemanticModel/` na raiz)
- Listar arquivos `.tmdl` em `./SemanticModel/tables/`
- Ler `./SemanticModel/model.tmdl`, `./SemanticModel/relationships.tmdl` (se existir)
- Contar: N tabelas, N medidas (varrer `measure` em todos os tmdl), N relacionamentos, tamanho do projeto

### 2. Rodar checks

Ler `references/checks.md` e rodar **as 6 famílias de check** descritas lá:

1. **Modelagem** — star schema, fact/dim, circular ref
2. **Relacionamentos** — cardinalidade, direção, ativo/inativo
3. **Performance** — colunas calculadas vs medidas, DISTINCTCOUNT em alta cardinalidade, etc
4. **Naming** — convenção, prefixos, consistência
5. **DAX** — DIVIDE vs `/`, variáveis não usadas, CALCULATE aninhado
6. **Documentação** — descrições ausentes, business keys

Cada check encontrado vira um **issue** com:
- `id` (slug único, ex: `model-flat-detected`)
- `categoria` (uma das 6 acima)
- `severidade` (`critical` | `medium` | `light`)
- `titulo` (uma frase, máx 90 chars)
- `path` (caminho do arquivo + linha quando aplicável)
- `por_que_importa` (1-2 parágrafos explicativos)
- `como_corrigir` (1 parágrafo + opcionalmente um snippet TMDL/DAX)

### 3. Calcular score

Fórmula simples:
```
score = max(0, 100 - (criticos × 4) - (medios × 1.5) - (leves × 0.4))
```

Limites:
- Score ≥ 85 → label "BOM" (gold)
- Score 65–84 → label "PRECISA ATENÇÃO" (gold)
- Score < 65 → label "CRÍTICO" (gold com glow mais forte)

A label sempre usa champagne gold (vibe v4 "alerta elegante", não "vermelho de painel").

### 4. Priorizar e ordenar

- Ordem: críticos primeiro, depois médios, depois leves
- Dentro de cada nível: ordenar por **categoria com mais issues** (concentração = prioridade de refator)

### 5. Gerar outputs

Filtrar issues conforme severidade mínima escolhida e gerar **dois arquivos**:

**A. Markdown** (versionável Git):
- Ler `templates/relatorio.md`
- Substituir placeholders pelos dados reais
- Salvar em `./_review/relatorio.md` na raiz do projeto Power BI

**B. HTML standalone** (visual):
- Ler `templates/relatorio.html`
- Substituir placeholders pelos dados reais
- Garantir que CSS continua **inline** (sem dependências externas além das fontes Google)
- Salvar em `./_review/index.html` na raiz do projeto Power BI

### 6. Resumir no chat

Devolver mensagem curta no chat com:
- Score + label
- Top 3 issues críticos (apenas títulos)
- Path dos 2 arquivos gerados
- Sugestão: "Abre `_review/index.html` no navegador pra ver completo"

## Outputs

```
[raiz do projeto Power BI do usuário]/
├── SemanticModel/                  ← input (não tocar)
├── Report/                         ← input (não tocar)
└── _review/                        ← OUTPUT da skill
    ├── relatorio.md                ← versão markdown (commitável)
    └── index.html                  ← versão visual standalone
```

## Edge cases

| Cenário | O que fazer |
|---|---|
| Modelo sem `.SemanticModel/` | Mensagem de pré-requisito (PBIP), encerra |
| Pasta `_review/` já existe | **Sobrescrever** (skill é idempotente) — mas avisar no chat |
| Modelo perfeito (zero issues) | Score 100 + celebrar + listar 3 melhores patterns encontrados (já que não tem o que reclamar) |
| Modelo apocalíptico (>50 críticos) | Truncar relatório nos top 30 críticos + linha "+ N issues não listados — começa por aqui antes" |
| Erro lendo um `.tmdl` específico (corrupção) | Reportar arquivo problemático no chat, seguir com os demais |
| Projeto enorme (>500 medidas) | Avisar "modelo grande, vou processar em chunks · ~5min" antes de começar |

## Idempotência

- Rodar 2x no mesmo projeto **sobrescreve** `_review/`
- Não cria backups, não acumula histórico (versionamento é via Git do usuário)
- Não modifica nada em `.SemanticModel/` ou `.Report/` — **somente leitura nesses**

## Segurança / Blast radius

- **Lê** arquivos do projeto Power BI (TMDL puro, texto)
- **Escreve** somente em `./_review/` (cria se não existe, sobrescreve se existe)
- **NÃO toca** em `.SemanticModel/`, `.Report/`, ou qualquer arquivo binário
- **NÃO commita** nada (segue regra inviolável git do CLAUDE.md raiz)
- **NÃO faz rede** — operação 100% local

## Tom do relatório

Estilo Xperiun (declarado em `contexto/posicionamento.md`):
- Direto, sem rodeio. Verdade > conveniência
- Tom de "colega sênior revisando código", não consultor formal
- Pode ser provocativo ("isso vai cobrar juros", "modelo plano é o anti-pattern #1")
- PT-BR com **todos os acentos** (regra inviolável CLAUDE.md)
- Usar metáforas concretas ("vai virar ligação às 7h da manhã" > "pode causar erro em produção")

Exemplos de **bom** vs **ruim**:

❌ Ruim: "Identificou-se a presença de relacionamento bi-direcional que pode impactar negativamente a performance do modelo."

✅ Bom: "Bi-direcional em N:1 com calendário causa ambiguidade de filtro e duplica scan no engine. Em modelo de 1.4GB você está pagando ~30% de overhead por refresh sem ganhar nada."

## Branding (footer e CTA do HTML)

O relatório HTML tem footer fixo:
- Linha: "Relatório gerado em 4 minutos por Claude Code + /pbi-modelo-review"
- CTA: "Quero usar esse skill no meu Power BI →" (link placeholder por enquanto)
- Meta: "XPERIUN · O Sistema Operacional dos Incomparáveis · pages.xperiun.com"

Branding é **sempre** Xperiun (não @leokarpa) — a skill faz parte do posicionamento institucional. Mesmo se o usuário rodar a skill no projeto dele, o footer mantém referência Xperiun (é nossa skill, ele só tá usando).

## Tempo típico

- Modelo pequeno (≤30 tabelas, ≤80 medidas): **1–2 min**
- Modelo médio (~50 tabelas, ~150 medidas): **3–5 min**
- Modelo grande (>200 medidas, >100 tabelas): **5–10 min**

Avisar ao usuário se > 3min esperados.

## Versão atual

`v0.1` — protótipo interno Xperiun OS. Ainda não distribuído publicamente.
Quando estabilizar, vira pacote no repo `xperiun/claude-code-powerbi-skills` (lead magnet).
