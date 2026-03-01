---
description: >
  Generate a spec for a feature or task. Includes objective, DoD,
  scope, anti-scope, technical decisions, and risks. Grounded in
  the project's real patterns from .vibeflow/.
  Usage: /vibeflow:gen-spec <feature description>
disable-model-invocation: true
---

## Idioma

TODO output DEVE ser em Português BR. A spec inteira deve ser escrita
em português, incluindo: objetivo, contexto, DoD, escopo, anti-escopo,
decisões técnicas e riscos. Termos técnicos em inglês são aceitáveis
quando são padrão da indústria (e.g. "endpoint", "middleware", "deploy").

Generate a complete spec for: $ARGUMENTS

## Before writing the spec:

0. Se $ARGUMENTS é um path para um arquivo `.md` dentro de `.vibeflow/prds/`,
   leia o PRD. Use o PRD como base para a spec — o problema, público,
   solução, escopo e anti-escopo já estão definidos. Foque em traduzir
   para decisões técnicas, DoD binário, e patterns aplicáveis.

1. Check if `.vibeflow/` exists. If it does:
   - Read `.vibeflow/index.md` for project context
   - Check for `Budget sugerido: ≤ N` line in index.md — use that as the
     budget for this spec. If not present, default to ≤ 6 files.
   - Read `.vibeflow/conventions.md` for coding standards
   - Read the pattern docs from `.vibeflow/patterns/` that are relevant
     to this feature
2. If `.vibeflow/` does NOT exist:
   - Warn the user: "No project analysis found. Run /vibeflow:analyze
     first for better results. Proceeding with direct code reading."
   - Read relevant files directly from the codebase
3. Identify what exists today related to this feature
4. Identify which existing patterns apply

## Then produce the spec:

- **Objective** — 1 sentence. What changes for the user.
- **Context** — What exists today and why this matters now.
- **Definition of Done** — 3-7 binary checks (pass/fail, no ambiguity).
- **Scope** — What's in.
- **Anti-scope** — What's explicitly OUT. Be aggressive.
- **Technical Decisions** — With trade-offs and justification.
- **Applicable Patterns** — List which patterns from `.vibeflow/patterns/`
  must be followed. If the feature introduces a NEW pattern, note it.
- **Risks** — Premortem: what can go wrong + mitigation.

Be opinionated. Cut scope aggressively. Challenge vague requirements.
If something is unclear, state your assumption and flag it with a TODO.

Save the spec to: `.vibeflow/specs/<feature-slug>.md`
Create the `.vibeflow/specs/` directory if it doesn't exist.

---

## Maintenance

If this command is modified, update `MANUAL.md` to reflect the changes.
