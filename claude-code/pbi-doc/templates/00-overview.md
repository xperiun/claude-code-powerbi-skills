<!--
  TEMPLATE · /pbi-doc · 00-overview.md
  Salvar como _docs/00-overview.md na raiz do projeto Power BI

  Placeholders globais (mesmos do template HTML):
    {{PROJECT_NAME}} {{PROJECT_FILENAME}} {{TIMESTAMP}} {{PROJECT_TAGLINE}}
    {{TABLES_COUNT}} {{MEASURES_COUNT}} {{RELATIONSHIPS_COUNT}} {{COLUMNS_COUNT}} {{SIZE}}

  Placeholders específicos:
    {{INVENTORY_TABLE_MD}}    — linhas markdown da tabela inventário
    {{DATA_SOURCES_LIST}}     — lista markdown de fontes
    {{WARNINGS_BLOCK}}        — blockquote de avisos (paths pessoais, etc) — pode ser vazio
    {{CONFIG_LIST_MD}}        — lista markdown de configurações
-->

# {{PROJECT_NAME}} — Overview

> **{{PROJECT_TAGLINE}}**
> Documentação gerada por Claude Code + `/pbi-doc` em {{TIMESTAMP}}

**Arquivo:** `{{PROJECT_FILENAME}}`

---

## Métricas

| Métrica | Valor |
|---|---|
| Tabelas reais | **{{TABLES_COUNT}}** (excluindo auto-date geradas) |
| Medidas | **{{MEASURES_COUNT}}** |
| Relacionamentos | **{{RELATIONSHIPS_COUNT}}** |
| Colunas totais | **{{COLUMNS_COUNT}}** |
| Tamanho .pbip | **{{SIZE}}** |

---

## Inventário de tabelas

| Tabela | Tipo | Colunas | Medidas | Source |
|---|---|---|---|---|
{{INVENTORY_TABLE_MD}}

> **Tipo:** Fato (1+ por modelo) · Dimensão (descreve fato) · Medidas (só hospeda DAX) · Aux (parameter table, calculated, etc.)

---

## Fontes de dados

{{DATA_SOURCES_LIST}}

{{WARNINGS_BLOCK}}

---

## Configurações relevantes do modelo

{{CONFIG_LIST_MD}}

---

## Próximas seções

- [01 · Tabelas](01-tabelas.md) — descrição + colunas tipadas + source M de cada tabela
- [02 · Medidas](02-medidas.md) — DAX completo + explicação PT de cada medida
- [03 · Relacionamentos](03-relacionamentos.md) — mapa visual + tabela detalhada
- [04 · Dependências](04-dependencias.md) — quem depende de quem entre as medidas

---

*XPERIUN · O Sistema Operacional dos Incomparáveis · `/pbi-doc`*
