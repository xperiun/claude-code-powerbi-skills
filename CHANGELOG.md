# Changelog

## 2026-04-29

- Sync de `pbi-doc`, `pbi-modelo-review`, `pbi-dax-create` com a versão local do xperiun-os.


Todas as mudanças notáveis nesse repo. Segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

---

## [Unreleased] · 2026-04-28

### Mudanças estruturais

- Reorganização: pastas das skills movidas pro diretório `claude-code/` (era root)
- Pasta `releases/` renomeada pra `claude-web/` (clareza pra audiência não-dev)
- Estrutura agora reflete os 2 caminhos de uso: `claude-code/` (clone+cp) e `claude-web/` (zip pra upload)

### Docs

- `INSTALL.md` atualizado com 3 caminhos (Web · Desktop · Code) e novos paths
- `README.md` enxugado · entry point + tabela comparativa · detalhes técnicos no INSTALL.md

---

## [0.1.0] · 2026-04-26 · Lançamento inicial

### Adicionado

- 3 skills core do Claude pra Power BI:
  - `pbi-modelo-review` · audita modelo (scorecard 0-100 + 23 checks em 6 famílias)
  - `pbi-doc` · gera mini-site HTML + 5 markdowns versionáveis
  - `pbi-dax-create` · cria medida DAX por descrição em PT
- Suporte a 3 ambientes Claude: Web (Free funciona), Desktop, Code
- Modo offline-friendly · skill detecta ambiente e adapta output
- LGPD-compatível · arquivos `.tmdl` ficam locais, só o que você anexa vai pra rede

### Roadmap

**v0.2 (planejado):**
- `pbi-dax-refactor` · refatora medidas existentes pro padrão SQLBI
- `pbi-perf` · auditoria focada em performance
- `pbi-from-csv` · gera modelo dimensional a partir de CSV/Excel
