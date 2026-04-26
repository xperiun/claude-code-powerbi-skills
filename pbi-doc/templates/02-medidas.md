<!--
  TEMPLATE · /pbi-doc · 02-medidas.md
  Salvar como _docs/02-medidas.md na raiz do projeto Power BI

  Placeholders:
    {{PROJECT_NAME}}      — Nome do projeto
    {{MEASURE_GROUPS_MD}} — Blocos markdown agrupados por displayFolder (ver padrão abaixo)

  ESTRUTURA: agrupar por displayFolder do TMDL. Sem folder vai em "Sem pasta".

  Padrão de cada GRUPO:

    ## {Nome do displayFolder}

    {N medidas · descrição curta do grupo}

    ### {Nome da medida 1}

    `{tabela host}.{medida}` · {formatString se relevante}

    **O que faz:**
    {1-3 frases em PT explicando o resultado da medida sem entrar em DAX. Foco no business meaning.}

    **DAX:**
    ```dax
    {DAX completo, formatado}
    ```

    **Como funciona:**
    {Explicação técnica em PT da lógica DAX. Se medida usa outras medidas, listar quais. Se usa funções time intelligence, explicar o contexto. Se tem CALCULATE, explicar o filter modifier.}

    **Usa:** {lista de outras medidas/colunas referenciadas}
    **É usada por:** {lista de medidas que dependem desta}

    ---

    ### {Próxima medida do grupo}
    ...

  ORDEM dentro de cada folder: alfabética.
  TOM: didático mas não condescendente — pra analista que sabe DAX, mas pode não conhecer o modelo específico.
-->

# {{PROJECT_NAME}} — Medidas

Catálogo de todas as medidas DAX do modelo, agrupadas pelos displayFolders.

> **Voltar pra:** [01 · Tabelas](01-tabelas.md) · **Próxima:** [03 · Relacionamentos](03-relacionamentos.md)

---

{{MEASURE_GROUPS_MD}}

---

*XPERIUN · `/pbi-doc`*
