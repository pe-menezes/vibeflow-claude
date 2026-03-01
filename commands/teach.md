---
description: >
  Teach the project knowledge base. Update .vibeflow/ docs with
  corrections, new conventions, decisions, or patterns based on
  natural language feedback.
  Usage: /vibeflow:teach <feedback>
disable-model-invocation: true
---

## Idioma

TODO output DEVE ser em Português BR. Termos técnicos em inglês
são aceitáveis quando são padrão da indústria.

Process feedback and update project knowledge: $ARGUMENTS

## Before starting

1. Check if `.vibeflow/` exists.
   - **YES** → read `.vibeflow/index.md` for orientation.
   - **NO** → warn: "`.vibeflow/` não existe. Rode `/vibeflow:analyze`
     primeiro para criar a knowledge base." and STOP.

## Classify the feedback

Read $ARGUMENTS and classify into one of these categories:

### (a) Correção de pattern existente
The user is saying an existing pattern doc is wrong or outdated.
- Identify which `patterns/*.md` file is affected.
- Read that file.
- Apply the correction OUTSIDE the `<!-- vibeflow:auto -->` markers
  (add a `## Manual Corrections` section at the end if it doesn't exist,
  or append to it).
- This ensures the correction survives incremental analyze runs.

### (b) Nova convenção
The user is adding a coding convention.
- Read `conventions.md`.
- Add the new convention OUTSIDE the `<!-- vibeflow:auto -->` markers
  (add a `## Team Conventions` section at the end if it doesn't exist,
  or append to it).

### (c) Decisão arquitetural
The user is recording an architectural decision.
- Read `decisions.md`.
- Add a new entry at the TOP (newest first), formatted as:

```
### <date> — <title>
**Decisão:** <what was decided>
**Contexto:** <why>
**Alternativas descartadas:** <what was not chosen and why>
```

### (d) Novo pattern
The user is describing a pattern that doesn't have a doc yet.
- Create a new file: `.vibeflow/patterns/<name>.md`
- Use the standard structure with markers:

```
# Pattern: <Name>

<!-- vibeflow:auto:start -->
## What
<from user feedback>

## Where
<inferred from feedback, or "A ser confirmado pelo analyze">

## The Pattern
<from user feedback — real code if provided, otherwise description>

## Rules
<from user feedback>
<!-- vibeflow:auto:end -->

## Anti-patterns
<from user feedback if mentioned, otherwise empty>
```

- Update `.vibeflow/index.md` → add the new pattern to "Pattern Docs Available".
- Update architect's MEMORY.md if it exists.

## After updating

Report to the user:
- What category was identified
- Which file(s) were modified
- What was added/changed (brief summary)
- Suggest: "Rode `/vibeflow:analyze` na próxima oportunidade para
  sincronizar as seções auto-geradas com suas correções."

## Rules

- NEVER modify content inside `<!-- vibeflow:auto:start/end -->` markers.
  User corrections go OUTSIDE markers to survive incremental updates.
- ALWAYS read the target file before modifying it.
- If the feedback is ambiguous, ask ONE clarifying question before acting.
- If `.vibeflow/` doesn't exist, STOP. Don't create it manually.

---

## Maintenance

If this command is modified, update `MANUAL.md` to reflect the changes.
