---
description: >
  Show statistics from audit reports. Reads all audits and compiles
  pass/fail rates, most violated patterns, and common DoD gaps.
  Usage: /vibeflow:stats
disable-model-invocation: true
---

## Idioma

TODO output DEVE ser em Português BR. Termos técnicos em inglês
são aceitáveis quando são padrão da indústria.

Compile and report statistics from audit reports.

## Steps

1. Check if `.vibeflow/audits/` directory exists and contains `.md` files.
   - If no audits found: report "Nenhum audit encontrado. Rode
     `/vibeflow:audit` após implementar uma feature." and STOP.

2. Read ALL `.md` files in `.vibeflow/audits/`.

3. For each audit file, extract:
   - **Verdict**: PASS, PARTIAL, or FAIL (from the `**Verdict:**` line)
   - **DoD checks**: count of passed `[x]` and failed `[ ]` checkboxes
     in the `### DoD Checklist` section
   - **Pattern violations**: items marked `[ ]` in the `### Pattern Compliance`
     section, with the pattern name
   - **Convention violations**: items listed in `### Convention Violations`
     section (if present)

4. Compile the statistics and report directly in the chat (do NOT save
   to file). Use this format:

```
## Vibeflow Stats

**Audits analisados:** N

### Verdicts
- PASS: N (X%)
- PARTIAL: N (X%)
- FAIL: N (X%)

### DoD
- Total de checks: N
- Taxa de aprovação: X%
- Checks que mais falham:
  1. "<check description>" — falhou N vezes
  2. "<check description>" — falhou N vezes
  3. "<check description>" — falhou N vezes

### Patterns
- Patterns mais violados:
  1. <pattern name> — N violações
  2. <pattern name> — N violações
  3. <pattern name> — N violações

### Convention Violations
- Total: N violações em N audits
- Mais comuns: <list top 3 if available>

### Tendência
<If ≥3 audits exist, note if quality is improving (more PASS over time),
stable, or degrading. Base on chronological order of audit dates.>
```

5. If there are fewer than 3 audits, skip the "Tendência" section.
   If there are no pattern violations, write "Nenhuma violação de pattern."

## Rules

- This command is READ-ONLY. It does NOT modify any files.
- Output goes directly to the chat, not to a file.
- Keep the report concise (~20-30 lines).
- If audit files have non-standard format, extract what you can and
  note: "Formato não-padrão detectado em <file>."

---

## Maintenance

If this command is modified, update `MANUAL.md` to reflect the changes.
