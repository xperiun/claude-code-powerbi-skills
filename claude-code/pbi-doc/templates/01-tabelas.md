<!--
  TEMPLATE · /pbi-doc · 01-tabelas.md
  Salvar como _docs/01-tabelas.md na raiz do projeto Power BI

  Placeholders:
    {{PROJECT_NAME}}     — Nome do projeto
    {{TABLES_BLOCKS_MD}} — Blocos markdown de cada tabela (ver padrão abaixo)

  Padrão de cada tabela (repetir TABLES_BLOCKS_MD pra cada):

    ## {Nome da tabela}

    > {Tagline 1 linha — papel + granularidade}
    > **Tipo:** {Fato | Dimensão | Tabela de medidas | Auxiliar} · **Origem:** {fonte resumida}

    ### Descrição
    {1-2 parágrafos. Se tabela tem `description:` no TMDL, usar. Senão, inferir do nome + colunas + uso em medidas.}

    ### Granularidade
    {1 linha = ?  — ex: "1 linha por item de NFe"}

    ### Colunas

    | Coluna | Tipo | Papel | Notas |
    |---|---|---|---|
    | `coluna1` | int64 | Chave primária | — |
    ...

    Onde **Papel** = um de: Chave primária / Chave estrangeira / Atributo / Métrica / Calculada
    E **Notas** = oculta, calculada, formato especial, etc.

    ### Source M (resumo)
    ```m
    let Fonte = ...
    in #"...":
    ```
    {Não copiar M inteiro se >15 linhas — resumir os passos relevantes}

    ---

  ORDEM: tabelas-fato primeiro, depois dimensões, depois auxiliares.
  EXCLUIR: tabelas LocalDateTable_* e DateTableTemplate_*.
-->

# {{PROJECT_NAME}} — Tabelas

Catálogo de tabelas do modelo. Cada uma com descrição, granularidade, colunas tipadas e source M resumido.

> **Voltar pra:** [00 · Overview](00-overview.md) · **Próxima:** [02 · Medidas](02-medidas.md)

---

{{TABLES_BLOCKS_MD}}

---

*XPERIUN · `/pbi-doc`*
