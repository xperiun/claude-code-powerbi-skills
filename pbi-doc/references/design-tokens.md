# Design Tokens · /pbi-doc (Xperiun v4)

Tokens canônicos do **Xperiun v4** (`design/xperiun-v4/design-system-manual-cultura.html`) usados pelo template HTML desta skill. Cópia local pra garantir que a skill é **standalone** (regra inviolável do CLAUDE.md raiz).

Mesmo conjunto de tokens da `/pbi-modelo-review` — vibe Xperiun v4 unificada nas skills Power BI. O que muda são os **componentes** específicos (sidebar, table-card, measure-card, etc.).

---

## Princípios visuais aplicados nesta skill

- **Vibe**: editorial premium · "manual técnico de luxo" tipo Linear docs / Stripe docs / Tailwind docs
- **Hierarquia**: tipografia condensada (Bebas Neue + Barlow Condensed) pra display, Outfit pra body, JetBrains Mono pra código
- **Cor primária**: blue glow (`--accent-glow: #7099FF`) pra elementos navegacionais e identificadores (paths, nomes de medidas)
- **Cor de destaque**: gold/champagne (`--accent-gold-bright: #E8C9A0`) pra valores numéricos e métricas-resumo
- **Tipo de tabela** (badge na sidebar e nos cards):
  - **Fato** → `--neon-magenta` (#C47FFF) — destaque, é o "núcleo" do modelo
  - **Dimensão** → `--accent-glow` (#7099FF) — informativo, descreve
  - **Tabela de medidas** → `--accent-gold-bright` (#E8C9A0) — especial, mora o cálculo
  - **Auxiliar** (parameter, aux, etc.) → `--text-muted` (#6B729A) — discreto, papel utilitário
- **Layout**: sidebar fixa esquerda (250px) + main scrollable. Diferente da `/pbi-modelo-review` (single column scrollable).
- **Background**: orb gradient fixo (navy/indigo) com radial blue overlay
- **Print**: vira light mode automático (preserva legibilidade no papel)

---

## Cores

### Backgrounds
```css
--bg-deepest:  #0D0C0E;
--bg-dark:     #100F14;
--bg-navy:     #1A1346;
--bg-indigo:   #1D164A;
--bg-card:     #14131A;
--bg-elevated: #1C1B28;
--bg-surface:  #211F30;
```

### Accent Blues
```css
--accent-deep:   #1A1346;
--accent-mid:    #3951A3;
--accent-blue:   #4668D4;
--accent-bright: #5A7FFF;
--accent-glow:   #7099FF;  /* PRIMÁRIO informativo desta skill */
```

### Accent Gold (champagne)
```css
--accent-gold-deep:   #B8955A;
--accent-gold:        #D4B896;
--accent-gold-bright: #E8C9A0;  /* PRIMÁRIO destaque desta skill */
--accent-gold-glow:   #F0DFC0;
```

### Neon secundários (tipos de tabela)
```css
--neon-cyan:    #4FE3E8;
--neon-magenta: #C47FFF;  /* TABELA FATO */
--neon-violet:  #7B6FFF;
--neon-teal:    #3FD1B8;
```

### Texto
```css
--text-white:     #FFFFFF;
--text-primary:   #F0EFF8;
--text-secondary: #9DA5C8;
--text-muted:     #6B729A;
--text-faint:     #3E4468;
```

### Borders
```css
--border-faint:  rgba(255,255,255,0.06);
--border-mid:    rgba(255,255,255,0.12);
--border-accent: rgba(90,127,255,0.35);
--border-gold:   rgba(212,184,150,0.35);
```

---

## Tipografia

```css
--font-display:   'Bebas Neue', sans-serif;
--font-condensed: 'Barlow Condensed', sans-serif;
--font-body:      'Outfit', sans-serif;
--font-mono:      'JetBrains Mono', Consolas, monospace;
```

### Carregamento (Google Fonts)
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Barlow+Condensed:wght@300;400;600;700;800&family=Outfit:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

### Escala
- `hero-headline`: `clamp(32px, 4vw, 48px)` (menor que /pbi-modelo-review — doc não tem scorecard hero)
- `section-title`: `clamp(28px, 3.5vw, 42px)`
- `card-title` (tabela/medida): `20px` fixo
- `code` (DAX/path): `13px` mono
- body padrão: `15px`
- sidebar nav: `13px`

---

## Spacing & layout

```css
--container-max: 1400px;        /* maior que /pbi-modelo-review por causa da sidebar */
--sidebar-width: 260px;
--gap-sm: 12px;
--gap-md: 24px;
--gap-lg: 40px;
--gap-xl: 80px;
--card-padding: 28px;
```

Padding lateral main content: `48px` (desktop) → `20px` (mobile).
Margin entre seções: `80px`.

### Layout fundamental

```
┌────────────────────────────────────────────────────────┐
│ TOPBAR (full-width, fixed)                             │
├──────────┬─────────────────────────────────────────────┤
│          │                                             │
│ SIDEBAR  │ MAIN CONTENT                                │
│ (fixed)  │ (scrollable)                                │
│ 260px    │                                             │
│          │ - Overview section                          │
│          │ - Tabelas section (cards)                   │
│          │ - Medidas section (grouped by displayFolder)│
│          │ - Relacionamentos section (matrix + list)   │
│          │ - Dependências section (tree + reverse)     │
│          │                                             │
│          │ FOOTER                                      │
│          │                                             │
└──────────┴─────────────────────────────────────────────┘
```

Em mobile (<780px): sidebar vira topbar dropdown ou hamburger.

---

## Radii

```css
--radius-sm:   6px;
--radius-md:   12px;
--radius-lg:   20px;
--radius-pill: 999px;
```

---

## Shadows

```css
--shadow-card:      0 4px 24px rgba(0,0,0,0.5);
--shadow-elevated:  0 8px 48px rgba(0,0,0,0.7);
--shadow-glow-sm:   0 0 20px rgba(90,127,255,0.3);
```

---

## Componentes específicos da /pbi-doc

### `.topbar`
Fixed top, full-width. Brand esquerda + nome do projeto centro + busca direita.

### `.sidebar`
Fixed left, 260px. Background `--bg-elevated`, scrollable. 5 seções principais (Overview/Tabelas/Medidas/Relacionamentos/Dependências) + sub-listas de tabelas/medidas (clicáveis pra anchor scroll).

### `.search-input`
Input no topbar. JS filtra cards por nome (busca client-side).

### `.metric-row` (overview)
Linha horizontal com 4-5 mini-cards: N tabelas | N medidas | N relacionamentos | N colunas | tamanho.

### `.table-card`
Card por tabela. Header com nome + badge tipo (fato/dim/aux). Body com descrição + granularidade + tabela de colunas tipadas. Source M colapsável.

### `.col-table`
Tabela HTML simples com colunas: Coluna, Tipo, Papel, Notas. Linha hover discreta.

### `.measure-group`
Container por displayFolder. Header com nome do folder + N medidas dentro.

### `.measure-card`
Card por medida. Nome em destaque, formatString abaixo. Pre com DAX colorizado. Box com explicação PT colapsável (`<details>`).

### `.dax-block` (substitui `.issue-body pre`)
Mesmo padrão de syntax highlight via classes:
- `.k` → keyword (magenta)
- `.f` → function/identifier/measure (blue glow)
- `.s` → string/number (gold)
- `.c` → comment (muted)

### `.rel-matrix`
Matriz visual ASCII art entre tabelas. Background dark, mono font, indica direção (↔, →) e cardinalidade.

### `.dep-tree`
Árvore de dependências indentada. Cada nível com bullet + linha vertical conectora.

### `.badge.tipo-fato` / `.tipo-dim` / `.tipo-medidas` / `.tipo-aux`
Badges pequenos pra indicar tipo de tabela. Cor segue mapeamento acima.

---

## Print stylesheet (obrigatório)

```css
@media print {
  body { background: white; color: #1A1A1A; }
  body::before, body::after { display: none; }
  .topbar, .sidebar { display: none; }
  .main { margin-left: 0; padding: 24px; }
  .table-card, .measure-card { box-shadow: none; background: white !important; border: 1px solid #ddd !important; break-inside: avoid; }
  .dax-block { background: #f5f5f5 !important; color: #1A1A1A !important; }
  .section-title, .card-title { color: #1A1A1A !important; }
  .search-input { display: none; }
}
```

---

## Regras invioláveis ao gerar HTML

1. **CSS sempre inline** no `<head>` — sem arquivos externos
2. **JS mínimo (vanilla)** — busca client-side + scroll spy + collapse de DAX longo (~40 linhas IIFE no fim do `<body>`). Sem framework.
3. **Fontes via Google Fonts CDN** — única exceção
4. **Responsive** — sidebar vira hamburger em <780px
5. **Print stylesheet sempre presente**
6. **Nunca inventar cores fora dos tokens**
7. **Footer Xperiun obrigatório**

## Versão

Tokens extraídos de `design/xperiun-v4/design-system-manual-cultura.html` em 2026-04-26. Última verificação de sincronia: 2026-04-26.
