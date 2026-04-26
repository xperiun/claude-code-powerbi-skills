# Template · Output da `/pbi-dax-create`

Formato da resposta no chat ao gerar uma medida nova. Esse template **não é um arquivo gerado** — é o **layout esperado da resposta** que a skill devolve no chat.

---

## Estrutura padrão (medida sem ambiguidade)

```markdown
## ✓ Medida pronta

**Nome sugerido:** `{Nome 1 — recomendado}`

> 3 sugestões de nome (escolha o que faz mais sentido pro time):
> 1. `{Nome 1}` — recomendado, segue padrão Title Case PT do projeto
> 2. `{Nome 2}` — alternativa mais explícita
> 3. `{Nome 3}` — alternativa mais curta

**O que faz:**
{1-3 frases business: o que essa medida calcula no domínio do negócio.}

**DAX:**
\`\`\`dax
{Nome recomendado} =
{DAX completo, formatado, com VAR/RETURN se >2 passos}
\`\`\`

**Como funciona:**
{Explicação técnica em PT linha-a-linha. Citar funções DAX usadas, contexto de filtro, dependências de outras medidas.}

**Onde colar:**
- Tabela host: `{TabelaMedidas}` (arquivo `tables/{TabelaMedidas}.tmdl`)
- displayFolder sugerido: `{Folder}` ({categoria, ex: temporal, padrão, dinâmica})
- Format string sugerido: `{format}` (ex: "R$ #,0;-R$ #,0;R$ 0")

**Posso aplicar direto?**
Quer que eu **edite o `.tmdl`** e adicione a medida agora, ou prefere **só copiar** e colar no Power BI Desktop manualmente?
```

---

## Variantes

### Quando há ambiguidade (precisa perguntar antes)

```markdown
Preciso de 1 esclarecimento antes de gerar:

Você descreveu **"{descrição vaga}"**. Pode ser entendido como:

a) **{Interpretação A}** — {DAX simplificado / contexto}
b) **{Interpretação B}** — {DAX simplificado / contexto}
c) **{Interpretação C}** (se aplicável)

Qual é o caso? (ou descreve melhor o que precisa)
```

### Quando coluna pedida não existe

```markdown
Você mencionou **`{coluna inexistente}`**, mas essa coluna não existe no modelo.

Encontrei essas colunas que podem ser o que você quer:
- `{tabela1}.{coluna1}` — {descrição inferida}
- `{tabela2}.{coluna2}` — {descrição inferida}
- `{tabela3}.{coluna3}` — {descrição inferida}

Qual é a correta? (ou descreve melhor o dado que você quer somar/contar)
```

### Quando medida similar já existe

```markdown
⚠️ Achei uma medida parecida no modelo:

**`{Nome existente}`** (em `Medidas.tmdl`, displayFolder `{folder}`):
\`\`\`dax
{DAX da medida existente}
\`\`\`

> {Frase comparando: "Faz quase a mesma coisa que você pediu, só usa X em vez de Y" / "É exatamente isso que você descreveu"}

**Opções:**
1. **Usar a existente** — copiar o nome e seguir
2. **Criar com nome diferente** — gera nova com variação (ex: filtro adicional, segmentação)
3. **Substituir a existente** — apago a antiga e crio essa nova

Qual?
```

### Quando aplicado direto (após confirmação)

```markdown
✓ Adicionada.

**`{Nome da medida}`** inserida em `Medidas.tmdl` (linha {N}, displayFolder `{folder}`).

Próximos passos:
1. Abre o **Power BI Desktop** — ele vai recarregar o modelo automaticamente
2. Verifica se a medida aparece no painel direito (pasta `{folder}`)
3. Testa em algum visual pra ver se calcula como esperado

Pra desfazer: `git diff tables/Medidas.tmdl` no terminal e descarte o que não quiser.

Quer criar mais alguma medida?
```

### Quando o pedido envolve antipattern (sinalizar antes de gerar)

```markdown
Antes de gerar, atenção:

Sua descrição me leva pra **{antipattern}** — pode ter problema de {performance / correção / leitura}.

**Alternativa recomendada:** {abordagem melhor}

**Quer que eu gere:**
1. ✓ A versão recomendada (mais limpa, sem o antipattern)
2. A versão exata que você pediu (com o antipattern, com aviso)

Default: vou pela 1 se não responder. Qual?
```

---

## Tom geral

- **Direto** — vai pra fórmula sem rodeio
- **Didático mas não condescendente** — assume que o usuário sabe DAX, só não conhece esse modelo específico
- **PT-BR com todos os acentos** (regra inviolável CLAUDE.md)
- **Provocativo quando útil** — sinaliza antipatterns sem ser chato
- **Sempre 3 sugestões de nome** quando a medida é gerada (não 1, não 5)
- **Sempre pergunta sobre aplicar direto vs copiar** ao final

---

## Code blocks

DAX em fence ` ```dax ` (renderiza com syntax highlight em chat web/IDE).

Em chat, evitar listas longas dentro do code block — DAX deve estar **autocontido e legível**.

---

## Versão

Template definido em 2026-04-26. Aplicado consistentemente em todas as gerações pra dar previsibilidade ao usuário (ele sabe o que esperar).
