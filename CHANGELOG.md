# Changelog

### v1.0.0 (2026-03-01)

- **Release estável.** Todas as 8 melhorias planejadas implementadas e verificadas.
- **Artefatos centralizados em `.vibeflow/`** — PRDs, specs, prompt packs e audits agora ficam todos dentro de `.vibeflow/` (antes eram pastas soltas na raiz do repo). Uma pasta só para commitar ou gitignore.
- `plugin.json` versão alinhada com MANUAL e CHANGELOG.
- Verify installation atualizado para listar todos os 8 commands.
- Budget contextual referenciado nos guardrails da skill (MANUAL).
- Consistência geral revisada e corrigida.

### v0.5.0 (2026-02-28)

- **Novo command: `/vibeflow:teach`** — feedback estruturado para atualizar `.vibeflow/`. Aceita correções de patterns, novas convenções, decisões arquiteturais e novos patterns em linguagem natural. Edita os docs fora dos markers auto para sobreviver a runs incrementais.
- **Novo command: `/vibeflow:stats`** — estatísticas de audits. Lê todos os audit reports e compila: taxa de PASS/PARTIAL/FAIL, patterns mais violados, DoD checks que mais falham, tendência de qualidade.
- **Budget contextual** — `/vibeflow:analyze` agora calcula um budget sugerido baseado no tamanho do projeto (2-3% dos source files, mín 4, máx 10) e grava em `.vibeflow/index.md`. `gen-spec` e `prompt-pack` leem esse valor. Fallback: ≤6 se não disponível.
- **Discover adaptativo** — `/vibeflow:discover` agora avalia clareza após a primeira resposta. Se problema, público e escopo já estão claros, usa fast-track (1-2 rodadas em vez de 3-5). Opening reformulado para convidar detalhamento upfront.
- **MANUAL.md enxugado** — changelog extraído para `CHANGELOG.md`. Seções condensadas: commands reference, workflow example, file map. Target ~500 linhas.

### v0.4.0 (2026-02-28)

- **Novo command: `/vibeflow:quick`** — fast-track para tasks pequenas. Um único comando que gera prompt pack direto, pulando discover e spec. Faz lightweight scan se `.vibeflow/` não existe. Budget padrão ≤4 arquivos. Spec efêmera (não persiste).
- **Architect model → Sonnet** — modelo padrão do agent architect mudou de Opus para Sonnet. ~3x mais barato e rápido, qualidade excelente para tarefas de spec-driven development. Opus disponível via edição manual do frontmatter em `agents/architect.md`.
- **Testes obrigatórios no audit** — `/vibeflow:audit` agora detecta e roda testes automaticamente baseado no stack do projeto (npm test, pytest, cargo test, etc.). Teste falhou = FAIL automático, independente do DoD. `/vibeflow:prompt-pack` agora sempre inclui seção de test commands obrigatórios.

### v0.3.0 (2026-02-26)

- `/vibeflow:analyze` — **incremental analysis:**
  - **Phase 0 (new)** — Detect Mode: checks if `.vibeflow/` exists and if `--fresh` flag is present. Routes to fresh or incremental mode. On incremental: reads previous analysis date, runs `git log` to find changed files, identifies affected modules. On "no changes": reports "Nenhuma mudança detectada" and exits early. Fallback to fresh if git unavailable.
  - **Phases 1-5 updated with incremental scoping:** Each phase now has an "Incremental mode:" paragraph describing what to preserve and what to re-analyze. Fresh mode unchanged.
  - **Marker system for pattern docs:** Pattern docs and `conventions.md` now use `<!-- vibeflow:auto:start/end -->` markers to delimit auto-generated sections. During incremental updates, only content within markers is regenerated; manual edits outside markers are preserved. Legacy pattern docs without markers are rewritten with markers added.
  - **decisions.md protection:** Never modified by analyze command in any mode (fresh or incremental). Created only on first run if doesn't exist. Reserved for architect and manual curation.
  - **Smart defaults:** `/vibeflow:analyze` (no args) runs incrementally if `.vibeflow/` exists. `--fresh` flag forces complete rebuild.
  - **Usage:** `/vibeflow:analyze [--fresh]`

### v0.2.0 (2026-02-26)

- `/vibeflow:analyze` — **enhanced to 6 phases with deep adaptive analysis and rules integration:**
  - Phase 1 now explicitly detects knowledge sources (`.cursorrules`, `.cursor/rules/*.mdc`, `CLAUDE.md`, `.clinerules`, `.github/copilot-instructions.md`, `/docs/`, `ARCHITECTURE.md`, `ADRs/`)
  - Phase 1.5 (new) — Rules Integration: extracts conventions from rules files, builds module map, validates rules against code, flags conflicts
  - Phase 2 (renamed from "Convention Mining") — adaptive sampling strategy: detects modules by directory/prefix/rules, reads ≥2 files per module, minimum 8 total, documents coverage gaps for large repos
  - Phase 3 (renamed from "Pattern Deep Dive") — expands scope using rules map; if a rule mentions a module not sampled, reads it for pattern docs
  - Phase 4 (renamed from "Compile") — `conventions.md` now incorporates rules-extracted conventions with source attribution; conflicts marked with ⚠️
  - Reporting now includes knowledge sources found, rules integrated, and conflicts detected

### v0.1.2 (2026-02-25)

- Novo command: `/vibeflow:discover` — diálogo interativo para PRD
- `gen-spec` aceita PRD como input (`/vibeflow:gen-spec prds/<slug>.md`)

### v0.1.1 (2026-02-25)

- Output padrão forçado para Português BR em todos os commands e agent

### v0.1.0 — Initial release

- `/vibeflow:analyze` — 5-phase adaptive codebase analysis, generates `.vibeflow/` pattern docs, updates architect MEMORY.md
- `/vibeflow:gen-spec` — grounded spec generation with Applicable Patterns section, reads `.vibeflow/` before writing
- `/vibeflow:prompt-pack` — self-contained prompt pack with real pattern code embedded, 8-section structure
- `/vibeflow:audit` — DoD + pattern compliance audit, incremental prompt pack on gaps, updates `decisions.md`
- **Architect agent** — `memory: project`, 2-layer knowledge system (MEMORY.md + `.vibeflow/`), maintains project knowledge across sessions
- **Skill: spec-driven-dev** — auto-loaded in planning contexts, enforces guardrails and methodology
- **`.vibeflow/`** — adaptive knowledge system committed to git, adaptive to any project type
