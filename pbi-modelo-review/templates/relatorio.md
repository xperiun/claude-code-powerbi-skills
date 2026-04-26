<!--
  TEMPLATE · /pbi-modelo-review · relatorio.md
  Versão markdown do relatório (versionável Git, lê em qualquer editor)

  Como usar:
  1. Substituir placeholders {{VAR}} pelos valores reais
  2. Repetir bloco {{ISSUE_BLOCK}} pra cada issue (manter a estrutura)
  3. Salvar como _review/relatorio.md na raiz do projeto Power BI

  Esse markdown é o "irmão" do index.html — mesmo conteúdo, mesma ordem, sem visual.
  Pra commit em Git, code review, leitura no editor.
-->

# Auditoria · {{PROJECT_NAME}}

> **Score: {{SCORE}}/100 — {{SCORE_LABEL}}**
> Gerado por Claude Code + `/pbi-modelo-review` em {{TIMESTAMP}}

**Modelo:** {{TABLES_COUNT}} tabelas · {{MEASURES_COUNT}} medidas · {{RELATIONSHIPS_COUNT}} relacionamentos · {{SIZE}}

---

## Veredicto

{{VERDICT_PLAIN}}

| Métrica | Valor |
|---|---|
| Issues totais | **{{TOTAL_ISSUES}}** |
| Críticos | **{{CRITICAL_COUNT}}** |
| Tempo estimado de correção | **{{TIME_TO_FIX}}** |

---

## Severidade

| Nível | Quantidade | Quando resolver |
|---|---|---|
| ▲ **Crítico** | {{SEV_CRITICAL_COUNT}} | Resolva primeiro — quebram refresh em produção, comprometem performance ou geram resultado errado |
| ● **Médio** | {{SEV_MEDIUM_COUNT}} | Próxima sprint — anti-patterns que pioram conforme modelo cresce |
| ○ **Leve** | {{SEV_LIGHT_COUNT}} | Quando der tempo — boas práticas, padronização, naming |

---

## Distribuição por categoria

{{DISTRIBUTION_TABLE}}

<!--
  Formato esperado de DISTRIBUTION_TABLE:
  | Categoria | Issues |
  |---|---|
  | Performance | 14 |
  | Relacionamentos | 11 |
  | ...
-->

---

## Issues priorizados

Issues abaixo estão **ordenados por severidade** (crítico → médio → leve) e **agrupados por categoria** (concentração = prioridade de refator).

{{ISSUES_BLOCKS}}

<!--
  Formato esperado de cada ISSUE_BLOCK:

  ### [SEVERIDADE] · [CATEGORIA] · {Título do issue}

  **Onde:** `path/do/arquivo.tmdl[:linha]`

  **Por que importa:**
  {Texto explicando por que isso é problema, adaptado ao contexto real do projeto.}

  **Como corrigir:**
  {Passo prático.}

  ```{tmdl|dax}
  // Snippet sugerido (opcional)
  ```

  ---

  Repetir o bloco pra cada issue.
-->

---

## Como rodar de novo

Quando quiser re-auditar (depois de aplicar correções, ou pra acompanhar evolução):

```
claude code  # na pasta raiz do projeto Power BI
> /pbi-modelo-review
```

A skill **sobrescreve** este relatório a cada execução. Pra acompanhar evolução, commite cada versão no Git e use `git diff _review/relatorio.md` pra ver o que mudou entre auditorias.

---

## Sobre essa skill

Auditoria gerada por **`/pbi-modelo-review`** — uma skill open-source da **Xperiun**, parte do toolkit Claude Code para Power BI.

- Operação 100% local · zero rede · zero XMLA · LGPD-compatível
- Lê apenas arquivos `.tmdl` (texto puro do PBIP)
- Não modifica nada em `SemanticModel/` ou `Report/` — somente leitura

Saiba mais: **[pages.xperiun.com](https://pages.xperiun.com)**

---

*XPERIUN · O Sistema Operacional dos Incomparáveis*
