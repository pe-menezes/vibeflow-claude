# Vibeflow Plugin — Manual

**Version:** 1.0.0
**Last updated:** 2026-03-01

> **Language:** Vibeflow outputs adapt to the language of the user's input.
> Technical terms in English are kept when they are industry standard.

This is the Single Source of Truth for the Vibeflow plugin.

---

## 1. What is Vibeflow?

Vibeflow is a plugin for Claude Code and Claude Cowork that implements **spec-driven development**. It bridges the gap between "idea" and "implementation" by forcing every task through a structured loop: analyze the codebase, write a grounded spec, generate a self-contained prompt pack for a coding agent, have the agent implement, then audit the result against a concrete Definition of Done and the project's own patterns.

The core loop:

```
/vibeflow:discover       ← dialogue → PRD (when the idea is vague)
         ↓
/vibeflow:analyze         ← run once (or when codebase changes significantly)
         ↓
/vibeflow:gen-spec        ← write a grounded spec (reads .vibeflow/; accepts PRD)
         ↓
/vibeflow:prompt-pack    ← generate self-contained pack for coding agent
         ↓
   coding agent implements ← Claude Code, Cursor, Copilot, etc.
         ↓
/vibeflow:audit          ← check DoD + pattern compliance
         ↓
    PASS? Ship it. PARTIAL/FAIL? incremental prompt pack → repeat
```

**Fast-track (small tasks):**

```
/vibeflow:quick "<description>"  ← generates prompt pack directly (skips discover/spec)
         ↓
   coding agent implements
         ↓
/vibeflow:audit               ← check DoD + pattern compliance + tests
```

---

## 2. Installation

### Option A: Local marketplace (development / self-hosted)

```bash
# 1. Create the marketplace wrapper directory
mkdir vibeflow-marketplace
cd vibeflow-marketplace
mkdir .claude-plugin

# 2. Write marketplace.json
cat > .claude-plugin/marketplace.json << 'EOF'
{
  "name": "vibeflow-marketplace",
  "owner": { "name": "Vibeflow" },
  "plugins": [{
    "name": "vibeflow",
    "source": "./vibeflow-plugin",
    "description": "Spec-driven development: analyze, spec, prompt pack, audit"
  }]
}
EOF

# 3. Clone (or symlink) the plugin repo
git clone https://github.com/pe-menezes/vibeflow.git vibeflow-plugin
# or: ln -s /path/to/vibeflow ./vibeflow-plugin

# 4. Register the marketplace in Claude Code
/plugin marketplace add /path/to/vibeflow-marketplace

# 5. Install the plugin
/plugin install vibeflow@vibeflow-marketplace

# 6. Restart Claude Code
```

### Option B: Upload directly to Cowork

1. Open the Cowork desktop app.
2. Go to **Customize → Browse Plugins → Upload Plugin**.
3. Select the `vibeflow` folder (root of this repo).
4. The plugin loads automatically — no restart needed.

### Verify installation

In Claude Code, run:

```
/help
```

You should see `vibeflow:discover`, `vibeflow:quick`, `vibeflow:analyze`, `vibeflow:gen-spec`, `vibeflow:prompt-pack`, `vibeflow:audit`, `vibeflow:teach`, and `vibeflow:stats` listed under available commands.

---

## 3. Commands Reference

### `/vibeflow:discover`

**Purpose:** Interactive dialogue to turn a vague idea into a clear, actionable PRD (Product Requirements Document).

**When to use:**
- You have a fuzzy idea or a broad area — "I want to improve the onboarding", "I need notifications".
- Scope is unclear and you want to sharpen it before writing a spec.
- You want someone to challenge your assumptions and cut scope before implementation.

**When NOT to use:**
- You already know exactly what you want → go straight to `/vibeflow:gen-spec`.

**Usage:**

```
/vibeflow:discover <ideia ou área>
```

**Examples:**

```
/vibeflow:discover "I want to improve the onboarding experience"
/vibeflow:discover "I need a notification system"
/vibeflow:discover "tests are slow and fragile"
```

**What it does:**
- Reads `.vibeflow/` (if present) to ground questions in the real project.
- Runs 3–5 rounds of focused questions: problem, audience, success criteria, scope, anti-scope.
- Challenges vague premises, forces decisions, cuts scope.
- When there is enough clarity (or after 5 rounds), generates a PRD and saves it to `.vibeflow/prds/<slug>.md`.

**Output:**
- PRD with: Problem, Target Audience, Proposed Solution, Success Criteria, Scope v0, Anti-scope, Technical Context, Open Questions.
- At the end, suggests running `/vibeflow:gen-spec .vibeflow/prds/<slug>.md` to move to a technical spec.

**Tip:** After the PRD is saved, run `/vibeflow:gen-spec .vibeflow/prds/<slug>.md` to produce a spec from it.

---

### `/vibeflow:quick`

**Purpose:** Fast-track for small tasks. Generates a prompt pack ready for a coding agent in a single command, skipping the full discover/analyze/spec pipeline.

**When to use:**
- Quick fixes or small features with clear requirements.
- You want a prompt pack NOW, not a paper trail.
- The task fits in ≤4 files.

**When NOT to use:**
- The idea is vague → use `/vibeflow:discover` first.
- You need full documentation for the team → use the full pipeline.
- The task is large or architecturally significant → use `/vibeflow:gen-spec`.

**Usage:**

```
/vibeflow:quick <task description>
```

**Examples:**

```
/vibeflow:quick "add dark mode toggle to settings"
/vibeflow:quick "fix mobile nav hamburger menu"
/vibeflow:quick "add loading spinner to data table"
```

**What it does:**
1. Checks if `.vibeflow/` exists. If yes, uses existing knowledge. If no, runs a lightweight scan (reads stack, structure, key files — does NOT generate `.vibeflow/`).
2. Generates an ephemeral spec in memory (objective, DoD with 3-5 checks, scope, anti-scope, budget ≤4 files). The spec is NOT saved to file.
3. Generates a self-contained prompt pack from the ephemeral spec, embedding real patterns from `.vibeflow/` if available.
4. Saves only the prompt pack to `.vibeflow/prompt-packs/<slug>.md`.

**Output:**
- Single file: `.vibeflow/prompt-packs/<feature-slug>.md` (self-contained, ready for coding agent).

**Budget:** Defaults to ≤4 files (tighter than standard ≤6). If the task needs more, the command warns and suggests using `/vibeflow:gen-spec` instead.

**Tips:**
- The spec is ephemeral — if you need it later for audit, use the full pipeline.
- If `.vibeflow/` doesn't exist, the prompt pack will be less grounded. Run `/vibeflow:analyze` first for richer results.
- After implementation, you can still run `/vibeflow:audit` — it works with any prompt pack.

---

### `/vibeflow:analyze`

**Purpose:** Deep-analyze the current codebase and build curated pattern documentation in `.vibeflow/`. Supports incremental analysis: if `.vibeflow/` already exists, detects changes via git and updates only affected modules.

**When to use:**
- At project start, before running any other Vibeflow command.
- After significant codebase changes (new modules, major refactors, new stack added) — runs incrementally by default.
- When the architect's pattern docs feel stale or incorrect (use `--fresh` flag to force a complete rebuild).
- Regularly during development — incremental runs preserve manual edits and are much faster.

**Usage:**

```
/vibeflow:analyze [--fresh]
```

- No arguments: runs incrementally if `.vibeflow/` exists, otherwise runs fresh.
- `--fresh` flag: forces a complete rebuild, replacing all `.vibeflow/` docs.

**What it does (7 phases):**

0. **Detect Mode** — checks if `.vibeflow/` exists and if `--fresh` flag is present. Routes to fresh or incremental mode. If incremental mode: reads the previous analysis date from `index.md`, runs `git log` to find changed files, filters out test/config files, and identifies affected modules. If no source files changed, reports "No changes detected" and exits. If git is unavailable, warns and falls back to fresh run.

1. **Discovery** — reads `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc.; inspects top-level directory structure (2-3 levels); searches for knowledge sources (`.cursorrules`, `.cursor/rules/*.mdc`, `CLAUDE.md`, `.clinerules`, `.github/copilot-instructions.md`, `/docs/`, `ARCHITECTURE.md`, `ADRs/`); reads `README` and existing architecture docs. Determines stack, project type, major structural units, and reports which knowledge sources were found. Does not assume any specific structure. In incremental mode: only re-detect sources that changed.

2. **Rules Integration** — reads all knowledge sources found in Phase 1. Extracts concrete conventions and modules/subsystems mentioned. Builds an internal map (not a final output) distinguishing between project-level rules (`.cursorrules`, `CLAUDE.md`) and domain-specific rules (`.cursor/rules/*.mdc`). Validates rules against code; flags conflicts (e.g., "⚠️ Conflict: .cursorrules says X, but code does Y"). This map guides the sampling strategy in Phase 3 and deep dive scope in Phase 4. In incremental mode: only re-extract rules from changed sources.

3. **Convention Mining (Adaptive Sampling)** — detects modules/domains automatically (by directory, file prefix, rule mentions). Reads ≥2 files per module, prioritizing largest files, most-imported files, and rule-mentioned files. Maintains minimum 8 files total, with no fixed ceiling. For repos >100 source files, documents coverage gaps. Documents naming conventions, file organization, imports, error handling, tests, state management, typing, logging. Focuses on repeated patterns, not one-offs. In incremental mode: only re-sample affected modules.

4. **Pattern Deep Dive** — for each significant pattern found, reads 3-5 real examples. Extracts the actual pattern (not theoretical), notes variations and edge cases. Expands scope to modules mentioned in the rules map (Phase 2) even if sampling didn't cover them. Creates one `.vibeflow/patterns/<name>.md` file per pattern with real code snippets. In incremental mode: only re-analyze affected patterns. Pattern docs use markers (`<!-- vibeflow:auto:start/end -->`) to preserve manual edits.

5. **Compile** — assembles `.vibeflow/index.md` (project overview, structural units, list of pattern docs, key files, critical deps, known tech debt), `.vibeflow/conventions.md` (dense, actionable rules with real examples, incorporating conventions from rules with source attribution like "via .cursorrules"), and `.vibeflow/decisions.md` (empty log on fresh run only, never touched in incremental runs). Marks rule-vs-code conflicts in conventions.md. In incremental mode: updates index with new analysis date, merges changed conventions within markers only, never modifies decisions.md.

6. **Update MEMORY.md** — writes a compact index (<200 lines) to the architect agent's MEMORY.md. Lists all `.vibeflow/` docs and includes a Quick Reference of the 5 most important conventions.

**What it generates:**

```
.vibeflow/
├── index.md          # Project overview, structural units, key files
├── conventions.md    # Coding conventions with real examples
├── patterns/
│   └── *.md          # One file per discovered pattern (adaptive)
└── decisions.md      # Architectural decision log (starts empty)
```

**Final report to the user includes:** stack detected, project type, structural units mapped, knowledge sources found and incorporated, pattern docs created/updated (listed), top 3 most critical patterns, conflicts detected (if any), any red flags or tech debt found. For incremental runs, also reports which modules were affected and what was updated.

**Tips:**
- Run from the root of the project you want to analyze, not from inside the plugin directory.
- Commit `.vibeflow/` to git. It's living documentation for the whole team.
- **Incremental by default:** Re-running analyze on a project with existing `.vibeflow/` runs incrementally, detecting changes via git and updating only affected modules. This is faster and preserves manual edits outside the `<!-- vibeflow:auto:start/end -->` markers.
- **Force rebuild with `--fresh`:** Use `/vibeflow:analyze --fresh` to completely rebuild `.vibeflow/`, replacing all docs. This is useful if the structure has dramatically changed or if you want to start fresh.
- **Manual edits:** Any changes you make outside the `<!-- vibeflow:auto:start/end -->` markers (e.g., in Anti-patterns sections, or in custom notes after the auto-generated content) will be preserved across incremental runs.
- The structure of `patterns/` is emergent — a monorepo KMP project will get `navigation.md`, `di.md`; a Next.js app might get `api-routes.md`, `data-fetching.md`; a Rust CLI might get `command-pattern.md`. The command adapts to what it finds.
- **Rules Integration:** The analyze command extracts conventions from your project's rules files (`.cursorrules`, `CLAUDE.md`, etc.) and incorporates them into `conventions.md` with source attribution. This ensures your curated pattern docs reflect both code reality and documented standards.
- **Markers preserve manual work:** Pattern docs and `conventions.md` use HTML comments (`<!-- vibeflow:auto:start/end -->`) to delimit auto-generated sections. Content outside markers is never touched during incremental updates.

---

### `/vibeflow:gen-spec`

**Purpose:** Generate a grounded spec for a feature or task, anchored to the project's real patterns from `.vibeflow/`.

**When to use:**
- Before starting any non-trivial feature or fix.
- When you need to clarify scope and Definition of Done before handing off to a coding agent.
- When a task feels vague and you need to force clarity.

**Usage:**

```
/vibeflow:gen-spec <feature description>
```

Examples:

```
/vibeflow:gen-spec "add dark mode toggle to the settings page"
/vibeflow:gen-spec "user authentication with OAuth2 and GitHub provider"
/vibeflow:gen-spec "replace REST polling with WebSocket for live updates"
```

**What it does:**

1. Checks if `.vibeflow/` exists.
   - If yes: reads `index.md` (project context), `conventions.md` (coding standards), and the pattern docs relevant to the feature.
   - If no: warns — *"No project analysis found. Run /vibeflow:analyze first for better results."* — then reads relevant files directly from the codebase.
2. Identifies what exists today related to the feature.
3. Identifies which existing patterns apply.
4. Produces the spec.

**Spec structure:**

| Section | Description |
|---------|-------------|
| **Objective** | 1 sentence. What changes for the user. |
| **Context** | What exists today and why this matters now. |
| **Definition of Done** | 3-7 binary pass/fail checks. No ambiguity. |
| **Scope** | What's in. |
| **Anti-scope** | What's explicitly OUT. Aggressive. |
| **Technical Decisions** | With trade-offs and justification. |
| **Applicable Patterns** | Patterns from `.vibeflow/patterns/` that must be followed. New patterns flagged. |
| **Risks** | Premortem: what can go wrong + mitigation. |

**What it generates:**

Saves to `.vibeflow/specs/<feature-slug>.md`. Creates `.vibeflow/specs/` if it doesn't exist.

**Tips:**
- The command is opinionated and cuts scope aggressively. If it disagrees with your framing, read why — it's usually right.
- Unclear requirements get flagged as TODOs. Resolve them before generating the prompt pack.
- The "Applicable Patterns" section is what connects the spec to `.vibeflow/`. If it's empty, run `analyze` first.

---

### `/vibeflow:prompt-pack`

**Purpose:** Generate a self-contained prompt pack from a spec, with real project patterns embedded, ready to hand off to any coding agent.

**When to use:**
- After a spec is approved and ready for implementation.
- When handing off work to a coding agent (Claude Code sub-agent, Cursor, Copilot, etc.).
- When you need to create a reusable, reproducible task description.

**Usage:**

```
/vibeflow:prompt-pack <spec-file-or-feature>
```

Examples:

```
/vibeflow:prompt-pack .vibeflow/specs/dark-mode-toggle.md
/vibeflow:prompt-pack "dark mode toggle"
/vibeflow:prompt-pack .vibeflow/specs/user-auth-oauth2.md
```

Pass a file path or a feature name — the command resolves it either way.

**What it does:**

1. Resolves the spec: reads the file if a path is given; searches `.vibeflow/specs/` if a name is given; generates a spec first if none exists.
2. Reads `.vibeflow/conventions.md` (always) and the pattern docs listed in the spec's "Applicable Patterns" section (or infers them if not listed).
3. Reads the actual codebase files relevant to the task.
4. Generates the prompt pack.

**Prompt pack structure (in order):**

| Section | Description |
|---------|-------------|
| Opening line | *"You are only seeing this prompt; there is no context outside it."* |
| **Objective + DoD** | Copied from spec. Non-negotiable. |
| **Anti-scope** | Copied from spec. |
| **Budget** | Max files to change (default ≤ 6). |
| **Patterns to Follow** | Real code examples from `.vibeflow/` embedded directly. **This is what makes it work.** |
| **Where to Work** | Verified real file paths + relevant code snippets for context. |
| **Directional Guidance** | Architectural direction, constraints. NOT step-by-step. |
| **How to Run/Test** | Validation commands. |
| **Docs to Update** | Which docs change after implementation. |

**What it generates:**

Saves to `.vibeflow/prompt-packs/<feature-slug>.md`. Creates `.vibeflow/prompt-packs/` if it doesn't exist.

**Tips:**
- Unverifiable file paths are flagged `<!-- TODO: verify this path -->`. Review before sending.
- Pattern docs from `.vibeflow/` are copied verbatim into the pack (including real code). The pack is self-contained — the coding agent doesn't need to read any other file.
- If patterns feel thin in the output, it means `.vibeflow/` is thin. Run `analyze` first.
- Prompt packs are agent-agnostic. You can paste them into any coding tool — Claude Code, Cursor, Copilot, ChatGPT Code Interpreter.

---

### `/vibeflow:audit`

**Purpose:** Audit a completed implementation against its Definition of Done AND the project's patterns from `.vibeflow/`.

**When to use:**
- After a coding agent finishes implementing a task.
- Before merging a PR when you want pattern compliance verified.
- When something feels off about an implementation but you can't pinpoint it.

**Usage:**

```
/vibeflow:audit <spec-file-or-feature>
```

Examples:

```
/vibeflow:audit .vibeflow/specs/dark-mode-toggle.md
/vibeflow:audit "dark mode toggle"
/vibeflow:audit .vibeflow/specs/user-auth-oauth2.md
```

**What it does:**

1. Finds the spec (from `.vibeflow/specs/` or provided path).
2. Extracts the Definition of Done.
3. Reads `.vibeflow/conventions.md` and pattern docs referenced in the spec (or inferred).
4. Reads the codebase files that were supposed to change.
5. Runs test/validation commands listed in the spec.
6. Audits **two things**: DoD compliance and pattern compliance.

**Audit report structure:**

```
## Audit Report: <feature>

**Verdict: PASS | PARTIAL | FAIL**

### DoD Checklist
- [x] Check 1 — evidence of compliance
- [ ] Check 2 — what's missing and why it fails

### Pattern Compliance
- [x] <pattern name> — follows correctly. Evidence: <file:line>
- [ ] <pattern name> — DEVIATION: <what's wrong, what it should be>

### Convention Violations (if any)
- <file> — <violation> — <what the convention says>

### Gaps (if PARTIAL or FAIL)
- What's missing / what's needed / estimated effort (S/M/L)

### Incremental Prompt Pack (if PARTIAL or FAIL)
A focused, self-contained pack covering ONLY the gaps.
```

**What it generates:**

- Saves audit report to `.vibeflow/audits/<feature-slug>-audit.md`. Creates `.vibeflow/audits/` if it doesn't exist.
- Updates `.vibeflow/decisions.md` if architectural decisions were made or new pitfalls discovered.

**Tips:**
- **Tests are mandatory.** The audit detects and runs tests automatically based on the project stack. If tests fail, verdict is FAIL regardless of DoD/pattern status.
- Verdicts are strict. If a DoD check or pattern cannot be verified from available context, it's marked FAIL with `"insufficient context to verify"` — never assumed compliant.
- On PARTIAL or FAIL, the incremental prompt pack is scoped to gaps only. Don't re-implement what already passes.
- The audit is the feedback loop that keeps `.vibeflow/` accurate — new pitfalls found during audits get written back to `decisions.md`.

---

### `/vibeflow:teach`

**Purpose:** Update `.vibeflow/` with corrections, new conventions, decisions, or patterns via natural language feedback.

**Usage:** `/vibeflow:teach <feedback>`

**Examples:**
```
/vibeflow:teach "our error handling pattern changed to Result<T>"
/vibeflow:teach "add convention: never use any in TypeScript"
```

**What it does:** Classifies feedback as (a) pattern correction, (b) new convention, (c) architectural decision, or (d) new pattern. Updates the appropriate `.vibeflow/` file OUTSIDE auto-generated markers so changes survive incremental analyze runs.

**Requires:** `.vibeflow/` must exist. Run `/vibeflow:analyze` first.

---

### `/vibeflow:stats`

**Purpose:** Compile statistics from audit reports. Shows pass/fail rates, most violated patterns, and quality trends.

**Usage:** `/vibeflow:stats`

**What it does:** Reads all files in `.vibeflow/audits/`, extracts verdicts, DoD check results, and pattern violations. Reports directly in the chat (no file saved). Shows trend if ≥3 audits exist.

---

## 4. Architect Agent

The **architect** is a sub-agent invoked automatically by Claude when the task involves planning, architecture, specs, or auditing. You don't call it directly — Claude delegates to it.

**Persona:** Senior software architect and technical PM. Thinks, plans, documents. Never implements.

**Tools available:** Read, Grep, Glob, Bash, WebSearch, WebFetch.

**Model:** Uses whatever model the user has configured. No model is hardcoded — the agent inherits the active model.

### Memory system (2 layers)

**Layer 1 — MEMORY.md (auto-loaded, <200 lines)**

Loaded automatically at session start. Contains:
- Project name, stack, date analyzed
- List of all `.vibeflow/` docs with 1-line descriptions
- Quick Reference: the 5 most important conventions to remember
- Instructions for the architect: read these docs before every task

The architect curates MEMORY.md to stay under 200 lines. If it grows beyond that, it summarizes and trims.

**Layer 2 — `.vibeflow/` (detailed, read on demand)**

The real knowledge. The architect reads relevant docs before every spec, prompt pack, or audit. After each task, it updates docs if something new was learned.

### Per-task workflow

1. Check MEMORY.md for orientation.
2. Read `.vibeflow/index.md` for project context.
3. Read relevant pattern docs for the task.
4. Use real patterns in output.
5. Update `.vibeflow/` if anything new was learned.

### What the architect does NOT do

- Does not write implementation code.
- Does not generate full source files.
- Does not validate bad ideas to be polite — it challenges them.
- If implementation is needed, it produces a prompt pack for a coding agent.

### Persistence between sessions

The `memory: project` flag means the architect's MEMORY.md persists across Claude Code sessions, scoped to the current project. In a new session, the architect reads MEMORY.md and immediately knows the project's stack, structure, and where the pattern docs are — without re-running analyze.

---

## 5. Project Knowledge (`.vibeflow/`)

### Structure

```
.vibeflow/
├── index.md          # Project overview, type, structure, key files, deps
├── conventions.md    # Coding conventions — dense, actionable, with examples
├── decisions.md      # Architectural decision log — newest first
├── patterns/
│   └── *.md          # One file per discovered pattern (varies by project)
├── prds/             # PRDs from /vibeflow:discover
├── specs/            # Specs from /vibeflow:gen-spec
├── prompt-packs/     # Prompt packs from /vibeflow:prompt-pack and :quick
└── audits/           # Audit reports from /vibeflow:audit
```

**Everything lives in `.vibeflow/`.** One folder to commit or gitignore as you prefer. If you want to ignore generated artifacts but keep the knowledge base:

```gitignore
# Keep knowledge, ignore artifacts
.vibeflow/prds/
.vibeflow/specs/
.vibeflow/prompt-packs/
.vibeflow/audits/
```

### How analyze generates it (adaptive)

The command never assumes a specific project type. It reads the code and creates pattern docs that match what it finds. Examples:

| Project type | Likely pattern docs |
|---|---|
| Monorepo (KMP) | `navigation.md`, `di.md`, `shared-models.md` |
| Next.js app | `api-routes.md`, `data-fetching.md`, `component-architecture.md` |
| Rust CLI | `command-pattern.md`, `error-handling.md` |
| Django API | `views.md`, `serializers.md`, `auth-flow.md` |
| React + Redux | `state-management.md`, `component-architecture.md`, `api-calls.md` |

### Pattern doc structure

Each file in `patterns/` follows this format:

```markdown
# Pattern: <Name>

## What
1-2 sentence description.

## Where
Which parts of the codebase use it.

## The Pattern
REAL code examples from this repo (not pseudocode).

## Rules
- Specific rules, naming conventions, dos/don'ts.

## Examples from this codebase
File: <real path>
<actual code snippet>

## Anti-patterns (if found)
Things in the codebase that break this pattern — marked as mistakes.
```

### How knowledge grows with use

- **`decisions.md`** grows as the architect makes decisions during specs and audits. New entries go at the top (newest first). The analyze command never modifies this file — it's reserved for manual and architect curation.
- **`patterns/*.md`** files are created or updated during incremental analysis. Content outside `<!-- vibeflow:auto:start/end -->` markers (like Anti-patterns sections and team notes) is preserved. The architect can also manually edit or add patterns between analyze runs.
- **`conventions.md`** is updated when conventions evolve. It incorporates conventions extracted from your rules files (`.cursorrules`, `CLAUDE.md`, `.cursor/rules/*.mdc`, etc.) with source attribution. Conflicts between rules and code are marked with ⚠️ signals. During incremental updates, only content within markers is regenerated; manual additions outside markers persist.
- **MEMORY.md** index is updated whenever new docs are added.
- **Incremental updates:** When you run analyze on a project with existing `.vibeflow/`, the command detects changes via git and updates only affected modules. This preserves manual edits and team discoveries that live outside the auto-generated sections.

### Why it goes in git

`.vibeflow/` is living documentation. The whole team benefits when it's committed:
- New team members onboard faster by reading `index.md` and `conventions.md`.
- Pattern docs prevent the "how did the last person do X?" question.
- `decisions.md` captures the "why" behind architectural choices.
- `.vibeflow/` is intentionally NOT in the plugin's `.gitignore`.

---

## 6. Skill: spec-driven-dev

The **spec-driven-dev** skill is loaded automatically by Claude when the conversation involves planning, architecture, or review — no command needed.

### What it teaches Claude

- **Core principle:** Never start coding without an objective (1 sentence), a Definition of Done (3-7 binary checks), explicit scope, explicit anti-scope, and a validation method.
- **Roles:** Claude = Architect (thinks, plans, challenges). Coding agent = Dev (implements, has no prior context).
- **Spec format:** Objective, Context, DoD, Scope, Anti-scope, Technical Decisions, Risks.
- **Prompt pack format:** Self-contained, starts with "You are only seeing this prompt", includes DoD, anti-scope, budget, patterns, real file paths, directional guidance, test commands, docs to update.
- **Audit format:** PASS / PARTIAL / FAIL verdict, DoD checklist with evidence, incremental prompt pack on gaps.

### Guardrails (enforced in all outputs)

- No DoD = no work.
- Minimum change to close the DoD. Nothing beyond.
- No refactoring "just because". No cleanup outside scope.
- Abstraction only with 2+ real uses.
- New dependency: justify in 1 line.
- Default budget: ≤ 6 files changed per task (or the contextual budget from `.vibeflow/index.md`, if available). Justify if exceeding.

### When to request more context before generating a prompt pack

- Touches DB / domain rules / critical calculations.
- Involves >1 route or >1 large component.
- Risk of exceeding >6 files.
- Bug without evidence (repro / logs / stack trace).

---

## 7. Workflow Example

Example using a fictional Next.js app:

```
# 1. (Optional) Discover — when the idea is vague
/vibeflow:discover "I want to add dark mode to settings"
# → answers questions → generates .vibeflow/prds/dark-mode-config.md

# 2. Analyze — run once per project (incremental after that)
/vibeflow:analyze
# → generates .vibeflow/ with patterns, conventions, index

# 3. Generate spec — from PRD or description
/vibeflow:gen-spec .vibeflow/prds/dark-mode-config.md
# or: /vibeflow:gen-spec "add dark mode toggle"
# → generates .vibeflow/specs/add-dark-mode-toggle.md

# 4. Generate prompt pack — self-contained for any coding agent
/vibeflow:prompt-pack .vibeflow/specs/add-dark-mode-toggle.md
# → generates .vibeflow/prompt-packs/add-dark-mode-toggle.md

# 5. Coding agent implements (paste prompt pack into any agent)

# 6. Audit — verify DoD + patterns + tests
/vibeflow:audit .vibeflow/specs/add-dark-mode-toggle.md
# → PASS? Ship. PARTIAL/FAIL? Use the incremental prompt pack and repeat.
```

**Fast-track alternative:** `/vibeflow:quick "add dark mode toggle"` → prompt pack in one step.

**Teaching the knowledge base:** `/vibeflow:teach "error handling now uses Result<T>"` → updates `.vibeflow/` docs.

**Checking progress:** `/vibeflow:stats` → shows audit pass rates and common violations.

---

## 8. File Map

| File | Purpose |
|------|---------|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `agents/architect.md` | Architect sub-agent (memory: project) |
| `skills/spec-driven-dev/SKILL.md` | Core methodology skill (auto-loaded) |
| `commands/discover.md` | Interactive dialogue → PRD |
| `commands/quick.md` | Fast-track → prompt pack |
| `commands/analyze.md` | Deep codebase analysis → `.vibeflow/` |
| `commands/gen-spec.md` | Grounded spec generation |
| `commands/prompt-pack.md` | Self-contained prompt pack |
| `commands/audit.md` | DoD + pattern + test compliance |
| `commands/teach.md` | Update `.vibeflow/` with feedback |
| `commands/stats.md` | Audit statistics |
| `MANUAL.md` | This file |
| `CHANGELOG.md` | Version history |

Generated files in the user's project are documented in section 5 (`.vibeflow/`).

---

## 9. Troubleshooting

**Commands not showing in `/help`**
Re-run `/plugin install vibeflow@vibeflow-marketplace` and restart Claude Code. If using Cowork, re-upload the plugin folder.

**`.vibeflow/` not found warning when running gen-spec or prompt-pack**
Run `/vibeflow:analyze` first from the project root. The warning message from the commands reads: *"No project analysis found. Run /vibeflow:analyze first for better results."*

**Manual edits lost after re-running analyze**
If your edits were inside `<!-- vibeflow:auto -->` markers, they will be overwritten during incremental updates. To preserve manual content: place it outside the markers (e.g., in the Anti-patterns section, or as new sections after the auto-generated content). Content outside markers is always preserved in incremental mode. If you're using an older pattern doc without markers, the entire doc will be rewritten with markers added on the next incremental run.

**Pattern docs are generic or vague**
The analyze command's adaptive sampling may have missed key modules. Check if your rules files (`.cursorrules`, `CLAUDE.md`, etc.) mention modules that weren't sampled — the command now expands scope based on rules in Phase 2. Alternatively, this happens in very small or scaffolded codebases. Run `/vibeflow:analyze` again after adding more code, or manually enrich the pattern docs — they're plain Markdown and can be edited at any time.

**Incremental analyze reports "no changes detected" but I know code changed**
This typically happens when changes are outside the source directory (e.g., only docs or test files changed, which are filtered by Phase 0). If you want to force an analysis that includes these changes, use `/vibeflow:analyze --fresh` to rebuild from scratch.

**MEMORY.md is growing too long (>200 lines)**
The architect curates it automatically on the next task. You can also edit it manually — it's a plain Markdown file at `.claude/agent-memory/architect/MEMORY.md`. Keep it under 200 lines: it's an index, not a knowledge base.

**Prompt pack doesn't embed real patterns**
Two causes: (1) `.vibeflow/` doesn't exist — run `analyze`. (2) The spec's "Applicable Patterns" section is empty or missing — edit the spec to list the relevant pattern files, or re-generate it after running `analyze`.

**Audit verdict seems too strict**
It is strict by design. If a DoD check or pattern can't be verified from code, it's FAIL. This is intentional — never assume compliance. Add evidence to the codebase (tests, comments, explicit code) so the next audit can verify.

---

## 10. FAQ

**Can I edit `.vibeflow/` manually?**
Yes. All files are plain Markdown. Edit pattern docs to add nuance, fix mistakes, or add patterns the analyze command missed. Commit your edits — they're part of your project's living documentation.

**Does `/vibeflow:analyze` overwrite existing `.vibeflow/`?**
By default, no — it runs incrementally. When you re-run analyze on a project with existing `.vibeflow/`, it detects what changed via git and updates only affected modules, preserving your manual edits (anything outside `<!-- vibeflow:auto:start/end -->` markers). Use `/vibeflow:analyze --fresh` to force a complete rebuild that replaces all `.vibeflow/` docs. With `--fresh`, if you want to preserve manual edits, copy them out beforehand and merge them back in afterward.

**Does the architect remember between sessions?**
Yes. The `memory: project` flag persists the architect's MEMORY.md scoped to the current project. In a new Claude Code session, the architect reads MEMORY.md and immediately knows the project's stack, structure, and where the pattern docs are.

**What if I don't run analyze before using other commands?**
`gen-spec`, `prompt-pack`, and `audit` still work — they fall back to reading the codebase directly. But they won't have the curated pattern knowledge, so outputs will be less grounded. The commands emit a warning: *"No project analysis found. Run /vibeflow:analyze first for better results."*

**Can I use prompt packs with Cursor, Copilot, or other agents?**
Yes. Prompt packs are plain Markdown and are agent-agnostic. Paste them into any coding tool. The self-contained format (opening disclaimer, embedded patterns, real file paths) was designed to work without any plugin infrastructure.

**Can I share `.vibeflow/` with my team?**
That's the intent. Commit it to git. When a teammate opens the project in Claude Code with Vibeflow installed, the architect reads the same `.vibeflow/` docs and works from the same grounding. Pattern knowledge becomes a team asset.

**What's the difference between the architect agent and just using the commands directly?**
Commands are invoked explicitly by you. The architect agent is a sub-agent Claude delegates to when it judges that planning or architecture work is needed. The architect also has persistent memory (MEMORY.md) and actively maintains `.vibeflow/` after each task. The commands don't have persistent memory — they're stateless instructions.

---

## 11. Changelog

See `CHANGELOG.md` for full version history.
