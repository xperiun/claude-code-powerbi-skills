# Instalação · Claude Skills Power BI

> 3 caminhos pra instalar as skills. Escolhe o que faz sentido pro seu setup.
> **TL;DR:** se tem dúvida, vai de **caminho 1 (Claude.ai web · Free)** — funciona pra 95% dos casos.

---

## Antes de começar — pré-requisitos

| Item | Por quê |
|---|---|
| **Conta Claude** (Free, Pro ou Max) | Pra rodar a skill |
| **Power BI Desktop** instalado | Pra abrir e salvar como PBIP |
| **Seu projeto Power BI salvo como `.pbip`** | Skills leem `.tmdl` (texto), não `.pbix` (binário) |

### Como salvar como PBIP (1 vez só, leva 30s)

1. Abre seu `.pbix` no Power BI Desktop
2. Menu `File → Save as → Power BI Project (.pbip)`
3. Vira uma **pasta** com 2 subpastas: `SemanticModel/` e `Report/`

A skill vai ler os `.tmdl` que estão dentro de `SemanticModel/definition/`. Tudo texto. Tudo local. Nada vai pra rede além do que você anexar manualmente no Claude.

---

## Caminho 1 · Claude.ai web (Free funciona) ⭐ recomendado

**Pra quem:** todo mundo que não tem Claude Code instalado. Funciona inclusive em conta **Free**.

**Tempo:** 2 minutos por skill.

### Passo 1 · Baixa o `.zip`

Vai na pasta [`releases/`](releases/) do repo e baixa o `.zip` da skill que você quer instalar:

| Skill | Arquivo |
|---|---|
| Auditar modelo | `pbi-modelo-review.zip` |
| Documentar modelo | `pbi-doc.zip` |
| Criar medida DAX | `pbi-dax-create.zip` |

> Pode baixar uma, duas ou as três. Cada uma é independente.

### Passo 2 · Sobe no Claude.ai

1. Abre [**claude.ai**](https://claude.ai) e faz login
2. Na sidebar esquerda, clica em **Personalizar** (ou ícone de engrenagem)
3. Clica em **Habilidades**
4. Clica em **+ Criar habilidade** → **Fazer upload de uma habilidade**
5. Arrasta o `.zip` que você baixou (ou clica e seleciona)

Pronto. A skill aparece em **"Habilidades pessoais"** e fica disponível em qualquer conversa nova.

### Passo 3 · Usa

Abre uma conversa nova e escreve, por exemplo:

```
audita esse modelo aqui
```

A skill vai pedir os arquivos `.tmdl`. Você anexa (arrastando do explorador ou via botão de anexo) e a skill processa.

**Quais arquivos anexar?**

Da pasta do seu projeto PBIP, vai em `SemanticModel/definition/` e anexa:
- `model.tmdl`
- `relationships.tmdl` (se existir)
- Todos os `.tmdl` dentro de `tables/` (1 por tabela)

> Dica: pode zipar a pasta `definition/` inteira e anexar 1 ZIP só. A skill descompacta.

### Limites do Free

- Claude Free tem **limite de mensagens / 5 horas** — modelo grande (>100 medidas) pode estourar antes de terminar
- Workaround: rodar 1 skill por vez, em conversa separada (limite reseta a cada 5h)
- Se rodar muito, vale Pro (R$ 100/mês) — limite ~5x maior

---

## Caminho 2 · Claude Desktop

**Pra quem:** já usa Claude Desktop (app nativo Mac/Windows). Mesmo passo a passo do Caminho 1, só muda o lugar onde sobe.

**Tempo:** 2 minutos por skill.

### Passo 1 · Baixa o `.zip`

Igual Caminho 1 — pega da pasta `releases/`.

### Passo 2 · Sobe no Claude Desktop

1. Abre o app **Claude Desktop**
2. Clica no seu **avatar** (canto superior direito) → **Configurações**
3. Vai em **Habilidades** (ou **Skills**)
4. **+ Criar habilidade** → **Fazer upload**
5. Arrasta o `.zip`

### Passo 3 · Usa

Igual Caminho 1 — abre conversa nova, escreve "audita esse modelo", anexa os `.tmdl`.

**Vantagem do Desktop sobre o Web:**
- Janela dedicada (não vive no navegador)
- Atalho de teclado pra abrir rápido
- Sincroniza com sua conta Pro/Max se tiver

---

## Caminho 3 · Claude Code (CLI/IDE)

**Pra quem:** desenvolvedor / analista que já tem Claude Code instalado. Roda **local**, lê arquivos do disco direto, salva output no projeto.

**Tempo:** 1 minuto por skill (cópia de pasta).

**Pré-requisitos extra:**
- Claude Code instalado (`npm i -g @anthropic-ai/claude-code` ou via VSCode extension)
- Conta **Claude Pro ou superior** (Free estoura limite rapidinho em modo Code)

### Passo 1 · Clona o repo (ou só essa pasta)

```bash
git clone https://github.com/xperiun/claude-code-powerbi-skills.git
cd claude-code-powerbi-skills
```

### Passo 2 · Copia as skills pra sua pasta de skills

**Modo global** (todas as conversas, qualquer projeto):

```bash
# Mac/Linux
cp -r .claude/skills/* ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse .claude/skills/* $env:USERPROFILE\.claude\skills\
```

**Modo por projeto** (só nesse projeto Power BI específico):

```bash
# Mac/Linux
cp -r .claude/skills/* /caminho/pro/seu-projeto-pbip/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse .claude/skills/* C:\caminho\pro\seu-projeto-pbip\.claude\skills\
```

### Passo 3 · Usa

Abre o Claude Code **dentro da pasta do seu projeto Power BI** (a que tem `SemanticModel/`):

```bash
cd /caminho/pro/seu-projeto-pbip
claude
```

Escreve direto:

```
audita esse modelo
```

A skill detecta a pasta `SemanticModel/` automaticamente, lê os `.tmdl` sozinha e salva o output em `_review/`, `_docs/` ou edita `Medidas.tmdl` direto (com aprovação).

**Vantagens do Code:**
- Lê pasta sozinha (não precisa anexar)
- Edita `.tmdl` direto (criação de medida vira 1 comando)
- Salva output versionável em Git
- Workflow contínuo (audita a cada release, doc viva)

---

## Comparativo dos 3 caminhos

| Aspecto | Web (Free) | Desktop | Code |
|---|---|---|---|
| Custo | **Grátis** | Grátis (Pro+ recomendado) | Pro+ obrigatório na prática |
| Setup | 2 min | 2 min | 5 min (CLI install) |
| Anexar arquivos | Toda conversa | Toda conversa | Lê sozinha |
| Edita `.tmdl` direto | ❌ (devolve bloco pra colar) | ❌ | ✅ |
| Salva output em disco | Download manual | Download manual | Auto em `_review/`, `_docs/` |
| Modelo grande (>200 medidas) | Pode estourar Free | OK em Pro+ | Melhor opção |
| Versiona no Git | Cópia manual | Cópia manual | Nativo |
| Limite por sessão | 5h Free / 5x Pro | Idem Web | Idem Web |

**Resumindo:**
- **Começou agora?** → Caminho 1 (Web Free)
- **Já usa Claude regularmente?** → Caminho 2 (Desktop) ou upgrade pra Pro
- **Tem workflow Power BI sério com Git?** → Caminho 3 (Code)

---

## Atualizar uma skill (quando sair versão nova)

### Web / Desktop
1. Vai em **Habilidades**
2. Clica nos **3 pontinhos** ao lado da skill antiga → **Excluir**
3. Sobe o `.zip` novo (mesmo processo do Passo 2 do Caminho 1)

### Code
```bash
cd claude-code-powerbi-skills
git pull
cp -r .claude/skills/* ~/.claude/skills/   # sobrescreve
```

---

## Desinstalar uma skill

### Web / Desktop
**Habilidades** → 3 pontinhos → **Excluir**

### Code
```bash
rm -rf ~/.claude/skills/pbi-modelo-review     # ou a skill que quer remover
```

---

## Troubleshooting

### "Não vejo o menu Habilidades no Claude.ai"
A feature de **Custom Skills** foi liberada pra contas Free entre Jan/Abr 2026. Se não aparece:
- Faz logout/login
- Tenta em janela anônima
- Confirma que tá em [claude.ai](https://claude.ai) (não claude.com)

### "Subi o .zip mas a skill não aparece nas conversas"
- Confere se a skill aparece em **Habilidades pessoais** (depois do upload)
- Em conversa nova, a skill ativa **automaticamente** quando você descreve a tarefa (ex: "audita esse modelo"). Não precisa selecionar manualmente.
- Se quiser garantir, escreve `/` no chat — uma lista de skills aparece

### "A skill pediu pra anexar `.tmdl` mas eu só tenho `.pbix`"
Você precisa salvar como PBIP primeiro:
- Power BI Desktop → `File → Save as → Power BI Project (.pbip)`
- Vira uma pasta · os `.tmdl` ficam em `SemanticModel/definition/`

### "Anexei os arquivos mas a skill diz que tá faltando algo"
Verifica que anexou:
- `model.tmdl` (raiz de `definition/`)
- `relationships.tmdl` (raiz de `definition/`, se houver relacionamentos)
- Todos os `.tmdl` de `definition/tables/` (1 por tabela), **excluindo** os arquivos `LocalDateTable_*.tmdl` e `DateTableTemplate_*.tmdl` (são auto-gerados, não precisa)

### "Modelo gigante, estourou o limite de mensagens"
- **Free**: aguarda 5h pro reset, ou roda 1 skill por vez
- **Pro/Max**: divide em chunks (ex: pra `/pbi-doc`, pede só "documenta a tabela fVendas" primeiro)
- Ou migra pro Caminho 3 (Code) que processa local sem limite de chat

### "Skill rodou mas o HTML quebrou no navegador"
- Baixa o artifact (botão download no Claude)
- Abre o `.html` direto no navegador (Chrome / Edge / Firefox)
- Se ainda quebra, abre uma issue no repo com o `.html` anexado

---

## Próximos passos

✅ Instalou? Bora usar.

- 🩺 **Audita primeiro** → roda `/pbi-modelo-review` no seu modelo principal. Vê o score.
- 📚 **Documenta depois** → `/pbi-doc` gera o handoff que você nunca teve tempo de fazer
- ⚡ **Cria medidas no fluxo** → `/pbi-dax-create` quando quiser pular o editor pequeno do Power BI Desktop

Dúvida? Issue no repo. Sugestão de skill nova? [Discussions](https://github.com/xperiun/claude-code-powerbi-skills/discussions).

---

*Construído com 🩵 pela equipe Xperiun. v0.1 · 2026-04*
