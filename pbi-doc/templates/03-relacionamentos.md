<!--
  TEMPLATE · /pbi-doc · 03-relacionamentos.md
  Salvar como _docs/03-relacionamentos.md na raiz do projeto Power BI

  Placeholders:
    {{PROJECT_NAME}}             — Nome do projeto
    {{REL_DIAGRAM_ASCII}}        — diagrama ASCII art (texto puro)
    {{REL_TABLE_MD_ROWS}}        — linhas markdown da tabela detalhada
    {{REL_ANALYSIS}}              — 1 parágrafo descritivo (sem opinar — descrever)

  Padrão do REL_DIAGRAM_ASCII (simplificado se modelo tem >10 tabelas, mostrar só fato + dims principais):

                      ┌─────────────┐
                      │ dCalendario │
                      └──────┬──────┘
                             │ 1:N
                             ▼
         ┌──────────────┐  N  ┌─────────┐  N  ┌──────────────┐
         │ FotoVendedor │◄────│ fVendas │────►│ FotoProduto  │
         └──────────────┘     └─────────┘     └──────────────┘
                                      (bi-direcional ⚠)

  Padrão de cada linha REL_TABLE_MD_ROWS:
    | 1 | `fVendas.cdProduto` | `FotoProduto.'Cod Produto'` | N:1 | **Bothdirections** | ✓ | Bi-direcional |
-->

# {{PROJECT_NAME}} — Relacionamentos

Mapa visual + lista detalhada de relacionamentos entre tabelas.

> **Voltar pra:** [02 · Medidas](02-medidas.md) · **Próxima:** [04 · Dependências](04-dependencias.md)

---

## Diagrama

```
{{REL_DIAGRAM_ASCII}}
```

---

## Tabela detalhada

| # | From | To | Cardinalidade | Direção | Ativo | Notas |
|---|---|---|---|---|---|---|
{{REL_TABLE_MD_ROWS}}

> **Notas:** sinalizar relacionamentos bi-direcionais, inativos, M:M, etc.

---

## Análise rápida

{{REL_ANALYSIS}}

> **Tom:** descrever, não opinar. Pra auditoria de qualidade dos relacionamentos (anti-patterns, bi-direcional desnecessário, etc.), use `/pbi-modelo-review`.

---

*XPERIUN · `/pbi-doc`*
