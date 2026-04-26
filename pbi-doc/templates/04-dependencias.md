<!--
  TEMPLATE · /pbi-doc · 04-dependencias.md
  Salvar como _docs/04-dependencias.md na raiz do projeto Power BI

  Placeholders:
    {{PROJECT_NAME}}            — Nome do projeto
    {{DEP_TREES_MD}}            — Árvores de medidas-raiz (uma ou mais)
    {{DEP_REVERSE_MD}}          — Cards reverse (medidas base com dependentes)
    {{TOP_TABLES_MD}}           — Lista markdown das tabelas mais referenciadas

  Padrão DEP_TREES_MD (árvore por medida-raiz, indentada):

    ### % Faturamento YoY
    ```
    % Faturamento YoY
    └─ Faturamento
    │  ├─ fVendas[QtdItens]
    │  └─ fVendas[PrecoUnitario]
    └─ Referência Faturamento LY
       ├─ Faturamento (já mapeada acima)
       └─ dCalendario[Data]
    ```

  Padrão DEP_REVERSE_MD (1 sub-section por medida-base):

    ### Faturamento (base)

    **Usada por:**
    - `Margem Bruta`
    - `% Faturamento YoY`
    - `Referência Faturamento LY`
    - `Medida Selecionada`

    **Implicação:** mudar `Faturamento` afeta 4 outras medidas. Cuidado em refator.

  Padrão TOP_TABLES_MD:
    1. `fVendas` — referenciada por X medidas
    2. `dCalendario` — referenciada por Y medidas
    3. ...
-->

# {{PROJECT_NAME}} — Dependências

Grafo de dependências entre medidas. Útil pra entender o impacto de mudanças e priorizar refator.

> **Voltar pra:** [03 · Relacionamentos](03-relacionamentos.md)

---

## Árvores por medida-raiz

Medidas-raiz são as que não são usadas por nenhuma outra. Mostram a árvore descendente até chegar nas colunas-folha.

{{DEP_TREES_MD}}

---

## Reverse — quem depende das medidas-base

Medidas-base são as mais reusadas no modelo. Mexer nelas afeta várias outras a jusante.

{{DEP_REVERSE_MD}}

---

## Tabelas mais referenciadas

Sinaliza onde mora a "carne" do modelo — tabelas usadas por mais medidas têm maior peso na arquitetura.

{{TOP_TABLES_MD}}

---

## Como usar essa info

- **Antes de refatorar uma medida-base** (ex: `Faturamento`), checa quem depende dela aqui — você pode estar quebrando 6 outras sem perceber
- **Em revisão de PR** que muda DAX, esse arquivo serve de "blast radius check"
- **Pra onboarding**, a árvore ajuda a entender o "esqueleto" de cálculo do modelo

---

*XPERIUN · `/pbi-doc`*
