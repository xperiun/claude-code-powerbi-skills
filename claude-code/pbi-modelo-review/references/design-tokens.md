# Design Tokens · /pbi-modelo-review (Xperiun v4)

Tokens canônicos do **Xperiun v4** (`design/xperiun-v4/design-system-manual-cultura.html`) usados pelo template HTML desta skill. Cópia local pra garantir que a skill é **standalone** (regra inviolável do CLAUDE.md raiz).

Se mudar tokens aqui sem mudar no template, fica inconsistente — sempre alterar nos dois lugares ou (melhor) re-extrair do canonical.

---

## Princípios visuais aplicados nesta skill

- **Vibe**: editorial premium (não neon). Tom "Bloomberg/FT", não "ferramenta de hacker"
- **Hierarquia**: tipografia condensada (Bebas Neue + Barlow Condensed) pra display, Outfit pra body
- **Cor primária**: blue glow (`--accent-glow: #7099FF`) pra elementos informativos
- **Cor de destaque**: gold/champagne (`--accent-gold-bright: #E8C9A0`) pra score, scores e CTAs
- **Severidade adaptada ao DS**:
  - Crítico → `--neon-magenta` (#C47FFF) — roxo violeta sutil (não vermelho agressivo)
  - Médio → `--accent-glow` (#7099FF) — blue glow
  - Leve → `--accent-gold-bright` (#E8C9A0) — gold sutil
- **Background**: orb gradient fixo (navy/indigo) com radial blue overlay
- **Brackets**: elemento característico do v4 — usar nos cards principais
- **Print**: vira light mode automático (preserva legibilidade no papel)

---

## Cores

### Backgrounds
```css
--bg-deepest:  #0D0C0E;  /* preto profundo, body base */
--bg-dark:     #100F14;
--bg-navy:     #1A1346;  /* navy do orb */
--bg-indigo:   #1D164A;  /* indigo do orb */
--bg-card:     #14131A;
--bg-elevated: #1C1B28;  /* surface elevado, cards */
--bg-surface:  #211F30;
```

### Accent Blues (orb palette)
```css
--accent-deep:   #1A1346;
--accent-mid:    #3951A3;
--accent-blue:   #4668D4;
--accent-bright: #5A7FFF;
--accent-glow:   #7099FF;  /* PRIMÁRIO informativo desta skill */
```

### Accent Gold (champagne — score, label, CTAs nobres)
```css
--accent-gold-deep:   #B8955A;
--accent-gold:        #D4B896;
--accent-gold-bright: #E8C9A0;  /* PRIMÁRIO destaque desta skill */
--accent-gold-glow:   #F0DFC0;
```

### Neon secundários (severity)
```css
--neon-cyan:    #4FE3E8;
--neon-magenta: #C47FFF;  /* SEVERIDADE CRÍTICA desta skill */
--neon-violet:  #7B6FFF;
--neon-teal:    #3FD1B8;
```

### Texto
```css
--text-white:     #FFFFFF;
--text-primary:   #F0EFF8;  /* body principal */
--text-secondary: #9DA5C8;  /* legendas, labels */
--text-muted:     #6B729A;  /* meta, footer */
--text-faint:     #3E4468;
```

### Borders
```css
--border-faint:  rgba(255,255,255,0.06);
--border-mid:    rgba(255,255,255,0.12);
--border-accent: rgba(90,127,255,0.35);  /* blue */
--border-gold:   rgba(212,184,150,0.35); /* gold */
```

---

## Tipografia

```css
--font-display:   'Bebas Neue', sans-serif;          /* h1, scorenum (uppercase, condensada) */
--font-condensed: 'Barlow Condensed', sans-serif;    /* labels, eyebrows, severity titles */
--font-body:      'Outfit', sans-serif;              /* body, cards, descriptions */
--font-mono:      'JetBrains Mono', Consolas, monospace; /* paths, snippets de código */
```

### Carregamento (Google Fonts — única dependência externa permitida)
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Barlow+Condensed:wght@300;400;600;700;800&family=Outfit:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

### Escala (fluida, mas não tão agressiva quanto slide)
- `score-num`: `clamp(7rem, 14vw, 12rem)`
- `hero-headline`: `clamp(40px, 6vw, 72px)`
- `section-title`: `clamp(36px, 5vw, 56px)`
- `sev-num`: `72px` fixo
- body padrão: `16px`
- labels uppercase: `11–13px`

---

## Spacing & layout

```css
--container-max: 1200px;
--gap-sm: 12px;
--gap-md: 24px;
--gap-lg: 48px;
--gap-xl: 80px;
--card-padding: 32px;
```

Padding lateral container: `40px` (desktop) → `24px` (mobile).
Margin entre seções: `96px`.

---

## Radii

```css
--radius-sm:   6px;   /* badges */
--radius-md:   12px;  /* mini-cards */
--radius-lg:   20px;  /* hero scorecard */
--radius-pill: 999px; /* buttons, badges */
```

---

## Shadows

```css
--shadow-card:      0 4px 24px rgba(0,0,0,0.5);
--shadow-elevated:  0 8px 48px rgba(0,0,0,0.7);
--shadow-glow-sm:   0 0 20px rgba(90,127,255,0.3);   /* hover blue */
--shadow-glow-md:   0 0 60px rgba(90,127,255,0.45);
--shadow-glow-gold: 0 0 24px rgba(232,201,160,0.25); /* score destaque */
```

---

## Gradients

```css
--gradient-orb: radial-gradient(ellipse at 38% 52%,
  #1F2E75 0%, #16205A 12%, #0F143E 28%, #0A0C24 48%, #0D0C0E 70%
);

--gradient-text: linear-gradient(135deg, #FFFFFF 0%, #A0B8FF 60%, #5A7FFF 100%);

--gradient-gold: linear-gradient(135deg, #F0DFC0 0%, #D4B896 55%, #B8955A 100%);

--gradient-surface: linear-gradient(135deg, #1C1B28 0%, #14131A 100%);
```

### Gradients por categoria do distribution-bar (cor por categoria)

| Categoria | Gradient |
|---|---|
| Performance | `linear-gradient(135deg, #5A7FFF, #4668D4)` |
| Relacionamentos | `linear-gradient(135deg, #7B6FFF, #4668D4)` |
| Modelagem | `linear-gradient(135deg, #C47FFF, #7B6FFF)` |
| Naming | `linear-gradient(135deg, #E8C9A0, #B8955A)` |
| DAX | `linear-gradient(135deg, #4FE3E8, #3FD1B8)` |
| Documentação | `linear-gradient(135deg, #6B729A, #3E4468)` |

---

## Componentes específicos da skill

Componentes novos criados pra essa skill (não existem no DS v4 base — extensões respeitando os tokens). Marcados com prefixo `pbi-` quando faria sentido modularizar; aqui usei nomes mais diretos pra simplicidade.

### `.scorecard`
Card hero do score. Background `--gradient-surface`, border `--border-faint`, radius `20px`, padding `64px`. Tem `::before` com gradient gold sutil no topo.

### `.score-num`
Texto gigante do score (0-100). Font condensed weight 300, color via `background-clip: text` com `--gradient-gold`, drop-shadow glow gold.

### `.score-label`
Pill com label ("Bom" / "Precisa Atenção" / "Crítico"). Sempre gold (semântica neutra elegante).

### `.sev-card`
Card de severidade (3 colunas). Variants `.critical`, `.medium`, `.light` mudam cor da `::before` line, do `.sev-num` e do `.sev-icon`.

### `.dist-bar`
Barra empilhada horizontal (flex). Cada `<div>` tem `flex` proporcional ao count e gradient da categoria.

### `.issue-card`
`<details>/<summary>` com header expansível. Variants `.critical` (border magenta) e `.medium` (border blue). Toggle visual de `+` rotaciona pra `×` quando aberto.

### `.issue-body pre`
Snippet de código com syntax highlight via spans:
- `.k` → keyword (magenta)
- `.f` → function/identifier (blue glow)
- `.s` → string (gold)
- `.c` → comment (muted)

---

## Print stylesheet (obrigatório)

```css
@media print {
  body { background: white; color: #1A1A1A; }
  body::before, body::after { display: none; }
  .scorecard, .sev-card, .issue-card, .dist-bar-container {
    box-shadow: none;
    background: white !important;
    border: 1px solid #ddd !important;
  }
  .score-num, .sev-num {
    color: #1A1A1A !important;
    -webkit-text-fill-color: #1A1A1A !important;
    filter: none !important;
  }
  .section-title, .issue-title { color: #1A1A1A !important; }
  .filters { display: none; }
}
```

Garante que ao imprimir/exportar PDF, o relatório vira light mode legível. Crítico pra circular em empresa.

---

## Regras invioláveis ao gerar HTML

1. **CSS sempre inline** no `<head>` — sem arquivos externos
2. **JS mínimo (vanilla)** — usar `<details>/<summary>` HTML nativo pra collapse de issues. Único JS necessário é o filtro de issues (~20 linhas vanilla, IIFE, no fim do `<body>`). Sem framework, sem build, sem dependência externa.
3. **Fontes via Google Fonts CDN** — única exceção de external CDN permitida
4. **Mobile-friendly** — testar em 375px (iPhone padrão) sem quebrar layout
5. **Print stylesheet sempre presente** — se faltar, relatório é inútil em ambiente corporativo
6. **Nunca inventar cores fora dos tokens** — se precisar de tom novo, derivar via `rgba()` dos tokens existentes
7. **Footer Xperiun obrigatório** — branding institucional não-negociável

## Versão

Tokens extraídos de `design/xperiun-v4/design-system-manual-cultura.html` em 2026-04-26. Última verificação de sincronia com o canonical: 2026-04-26.
