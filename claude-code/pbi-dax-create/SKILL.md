---
name: pbi-dax-create
description: Cria medida DAX a partir de descrição em PT, respeitando o modelo Power BI existente (colunas reais, naming convention, padrões DAX). Use quando o usuário pedir "cria medida pra X", "preciso de DAX que faça Y", "quero calcular Z", "uma fórmula pra...", ou qualquer pedido pra gerar nova medida em projeto PBIP aberto.
---

# /pbi-dax-create — Criação de medida DAX

Gera medidas DAX a partir de descrição em linguagem natural, **respeitando o modelo existente**:
- Usa as colunas e tabelas que **realmente existem** (sem inventar)
- Detecta o **padrão de nomenclatura** do projeto (Title Case, camelCase, com prefixos, etc.) e segue
- Aplica **boas práticas DAX** automaticamente (DIVIDE em vez de /, SUMX vs SUM correto, time intelligence apropriado)
- Avisa se a medida pedida **já existe** ou tem similar parecida
- Faz **pergunta clarificadora** quando o pedido é ambíguo

**Output principal**: medida DAX no chat com nome, fórmula completa, explicação linha-a-linha e onde colar. **Opcional**: aplica direto no `.tmdl` (com aprovação explícita).

## Quando usar

- Analista que sabe o que quer mas quer pular o "digitar DAX no editor"
- Líder de dados criando medidas em batch sem abrir Power BI Desktop
- Refatorando uma medida pra padrão DAX recomendado
- Onboarding: aprendiz descreve em PT e a skill mostra a forma correta

**Não usar quando:**
- Quer documentar medidas existentes → use `/pbi-doc`
- Quer auditar qualidade de modelo → use `/pbi-modelo-review`
- Quer refatorar medidas existentes em batch (skill futura `/pbi-dax-refactor`)
- Quer apenas explicar um DAX que já existe (use `/pbi-doc` que tem isso)

## Pré-requisitos

1. **Projeto em formato PBIP** (Power BI Project) — pasta com `.SemanticModel/`. Se faltar, instruir conversão e encerrar.
2. Acesso aos arquivos `.tmdl` (via filesystem ou upload — ver "Modos de execução" abaixo)

Se faltar PBIP, mensagem curta:
> Esse projeto ainda está em `.pbix` (binário). Pra eu criar medidas com base no modelo real, salva como Power BI Project: `File → Save as → Power BI Project (.pbip)`. Sem isso eu não consigo ler colunas/medidas existentes pra te dar uma medida que funciona.

## Modos de execução

A skill detecta automaticamente o ambiente e adapta input/output:

### Modo Code (Claude Code · Desktop · file-based)
- **Detecção**: tenho acesso a filesystem e a pasta atual contém `.SemanticModel/`
- **Input**: leio automaticamente os `.tmdl` da pasta
- **Output**: medida no chat. Se aprovado, **edito direto** o `.tmdl` da tabela host

### Modo Web (Claude.ai · upload-based)
- **Detecção**: não tenho acesso a filesystem (claude.ai web)
- **Input**: peço ao usuário pra anexar os arquivos:
  > Pra eu criar uma medida que funciona com o seu modelo real, anexe nesse chat:
  > - `model.tmdl` (lista geral de tabelas)
  > - O `.tmdl` da tabela onde a medida vai ficar (ex: `Medidas.tmdl`)
  > - Se a medida vai usar colunas/medidas de outras tabelas, anexa também esses `.tmdl`
  >
  > Pode arrastar individualmente ou zipar tudo num arquivo só.
- **Output**: medida no chat (3 nomes + DAX + explicação + onde colar). Como não posso editar arquivo direto no web, devolvo o **bloco TMDL pronto** pra usuário copiar e colar manualmente no arquivo (ou no Power BI Desktop).

### Detecção automática

Verificar se a pasta `.SemanticModel/` é acessível via filesystem:
- ✅ Sim → Modo Code (file-based, edição direta possível)
- ❌ Não → Modo Web (peço uploads, devolvo bloco pra copiar)

Se ambíguo, perguntar uma vez:
> Você tá rodando isso no **Claude Code** (CLI/IDE com acesso à pasta) ou no **claude.ai** (web)? Pra Code eu posso editar o `.tmdl` direto; pra web você precisa colar o bloco que eu te mando.

## Inputs

- **Descrição em PT** (obrigatório) — o que a medida deve calcular
- **(Opcional) Tabela host** — onde colocar. Default: tabela `Medidas` se existir, senão pergunta

Se descrição for vaga ou ambígua (ex: "vendas totais" — bruto ou líquido?), **fazer 1 pergunta clarificadora antes** de gerar.

## Processo

### 1. Detectar e mapear o modelo

- Confirmar `.SemanticModel/`
- Ler `model.tmdl` (config), `relationships.tmdl`, `tables/*.tmdl`
- Ignorar `LocalDateTable_*` e `DateTableTemplate_*` (auto-date hidden)
- Inventariar:
  - Tabelas (e qual delas é "Medidas" container, se existir)
  - Todas as colunas tipadas (qual tabela, tipo, papel inferido)
  - Todas as medidas existentes (nome, DAX, displayFolder)
  - Padrão de naming detectado nas medidas (ver `references/padroes.md`)

### 2. Interpretar a solicitação

Da descrição em PT, identificar:
- **Tipo de cálculo**: agregação simples (soma/média/contagem), divisão (%, ratio), comparação temporal (YoY, MTD, YTD), ranking, filtro condicional, lógica IF/SWITCH
- **Colunas/medidas necessárias**: tem que existir no modelo. Se a descrição menciona algo que não existe → sugerir alternativas reais
- **Granularidade implícita**: se "total" provavelmente quer SUM/SUMX, se "média" AVERAGE, se "único/distintos" DISTINCTCOUNT, etc.

**Se ambíguo**, parar e perguntar (1 pergunta única, objetiva). Exemplos:
- "vendas totais" → "Total de qual coisa? (a) Receita bruta com SUMX(Vendas, Qtd × Preço), (b) Receita líquida com algum desconto/devolução, (c) Quantidade total de itens vendidos"
- "% de margem" → "Sobre o quê? (a) Margem Bruta / Faturamento, (b) Margem Líquida / Receita Líquida"

### 3. Aplicar padrões DAX

Ler `references/padroes.md` e seguir as regras:
- `DIVIDE(num, den)` em vez de `num / den` (evita ∞ e erros silenciosos)
- `SUMX(tabela, expressão)` quando expressão envolve mais de 1 coluna por linha
- `SUM(tabela[col])` quando é soma simples de 1 coluna
- `DISTINCTCOUNT()` em coluna inteira (key) quando possível
- Time intelligence (`SAMEPERIODLASTYEAR`, `DATEADD`, `DATESYTD`) sempre referenciando coluna de tabela calendário
- `CALCULATE` com filter modifier explícito quando precisa modificar contexto
- `VAR ... RETURN` pra medidas com 3+ passos lógicos (legibilidade)

### 4. Aplicar padrão de nomenclatura

Detectar o padrão usado nas medidas existentes do projeto (ver `references/padroes.md` seção naming):
- **Title Case PT** (`Total Vendas`, `% Margem Bruta`)
- **Prefixo de tipo** (`% Faturamento YoY`, `# Pedidos`, `Δ MoM`)
- **camelCase** (`totalVendas`)
- **Mistura** (sinalizar que tá inconsistente, sugerir o mais comum)

Sempre gerar 3 sugestões de nome seguindo o padrão dominante.

### 5. Gerar a medida

Verificar antes de retornar:
- ✅ Todas as colunas/medidas referenciadas existem no modelo
- ✅ DAX segue boas práticas
- ✅ Nome segue padrão do projeto
- ⚠️ Existe medida similar no modelo? (busca por nome parecido OU mesma fórmula) → avisar antes de duplicar

### 6. Retornar no chat

Usar o formato definido em `templates/output-medida.md`. Resposta:
- 3 sugestões de nome (1ª recomendada, com justificativa)
- DAX completo formatado
- Explicação linha-a-linha em PT
- Onde colar (qual `.tmdl`, em qual seção/displayFolder)
- displayFolder sugerido (se aplicável)

### 7. Aplicação opcional

Ao final, perguntar:
> Quer que eu **aplique direto** no `Medidas.tmdl` (vou editar o arquivo) ou prefere **só copiar** e colar no Power BI manualmente?

**Default seguro**: só mostra. Aplica direto somente se o usuário disser sim explicitamente.

Se aplicar:
- Localizar a tabela host no `.tmdl`
- Inserir a medida na seção correta (alfabética dentro do displayFolder)
- Adicionar `description:` opcional se útil
- Salvar
- **Não commita** (regra inviolável git do CLAUDE.md raiz)

## Outputs

**Modo padrão** (só sugere):
- Resposta no chat com nome + DAX + explicação + instruções de colar
- Nenhum arquivo modificado

**Modo aplicar** (com aprovação explícita):
- `.tmdl` da tabela host modificado (medida inserida)
- Resumo curto no chat: "Medida X adicionada em Medidas.tmdl, linha Y"

## Edge cases

| Cenário | O que fazer |
|---|---|
| Descrição muito vaga | 1 pergunta clarificadora antes de gerar |
| Coluna pedida não existe | Sugerir as 2-3 colunas reais mais próximas + perguntar qual |
| Medida pedida já existe (mesmo nome) | Avisar e perguntar: substituir ou criar com nome diferente? |
| Medida com fórmula similar já existe | Avisar e mostrar a existente — perguntar se é o que ele quer ou se realmente quer criar nova |
| Sem tabela "Medidas" no modelo | Listar tabelas e perguntar onde colocar (ou sugerir criar tabela vazia "Medidas") |
| Time intelligence sem dim Calendario marcada | Avisar limitação + criar a medida mas sinalizar que pode dar resultado errado |
| Modelo enorme (>500 medidas) | Carregar só nomes+DAX-essência das existentes (não DAX completo) pra evitar contexto explodir |

## Tom

Estilo Xperiun:
- **Direto**: vai pra fórmula sem rodeio teórico
- **Didático mas não condescendente**: explica linha-a-linha em PT, mas no nível de quem já sabe DAX
- **Provocativo quando útil**: se o pedido tem antipattern óbvio, sinaliza ("você quer SUMX direto em fato de 5M linhas? Pode dar trabalho — alternativa: ...")
- **PT-BR com todos os acentos** (regra inviolável CLAUDE.md)

Exemplos:

❌ Ruim: "Para calcular o faturamento, podemos utilizar a função SUMX que iterará..."

✅ Bom: "**Faturamento** — SUMX em fVendas multiplicando QtdItens × PrecoUnitario. Mais limpo que coluna calculada (que infla a fato)."

## Segurança / Blast radius

- **Lê** arquivos `.tmdl` (texto puro)
- **Modo padrão**: zero modificação no projeto
- **Modo aplicar**: edita só o `.tmdl` da tabela host, nunca toca outros
- **NÃO commita** (segue regra inviolável git)
- **NÃO faz rede** — operação 100% local

## Tom dos avisos sobre risco

Quando aplicar direto:
- Antes: "Vou editar `Medidas.tmdl` e adicionar a medida na seção `0. Medidas Padrão`. Pode?"
- Após: "Adicionada. Abre o Power BI Desktop e ele vai recarregar o modelo automaticamente. Se quiser desfazer, só rodar `git diff` e descartar."

## Tempo típico

- Medida simples (agregação ou divisão): **15–30s**
- Medida com time intelligence ou múltiplos VAR: **30–60s**
- Medida que requer pergunta clarificadora: **+ tempo de ida-e-volta** com você

## Versão

`v0.1` — protótipo interno Xperiun OS. Faz parte do trio do lead magnet (`/pbi-modelo-review` · `/pbi-doc` · `/pbi-dax-create`).
