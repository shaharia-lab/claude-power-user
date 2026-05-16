# Quality Spec for AI-Context Files

Opinionated definition of what a healthy AI-context file looks like. Used by `agent-context-sync` in the lint phase. Each section below maps to one or more checks the skill performs.

> **Principle:** The audience is an AI assistant working on this codebase, not a human reading on a Sunday afternoon. Content that a competent AI could infer from a 30-second grep is **noise**, not documentation. Optimize for **claims the AI cannot easily derive from the repo itself**.

---

## Required sections

### Markdown files (CLAUDE.md, AGENTS.md, .github/copilot-instructions.md)

| Section | Min length | Required? | Notes |
|---------|------------|-----------|-------|
| **Project Overview** | 1 sentence | Yes | What this repo is, in one breath. |
| **Architecture** | 3 lines | Yes | Only non-obvious structure. Skip if the repo has <5 source files. |
| **Commands** | 3 lines | Yes if any exist | Build, test, lint, run. Omit only if the repo truly has none (rare). |
| **Conventions** | 3 lines | Recommended | Project-specific norms (commit format, naming, layout). |
| **Gotchas / What Not to Do** | 1 line | Recommended | High-value land mines. |

Detection rule: a section is "present" if there is a heading (any level) whose normalized text contains one of the canonical phrases below, OR a structurally equivalent block in the first 80% of the file.

| Section | Heading aliases |
|---------|-----------------|
| Project Overview | "overview", "about", "what is this", "introduction", "project" |
| Architecture | "architecture", "structure", "layout", "how it's organized", "components" |
| Commands | "commands", "build", "scripts", "tasks", "development workflow", "running" |
| Conventions | "conventions", "style", "standards", "rules", "patterns" |
| Gotchas | "gotchas", "what not to do", "pitfalls", "warnings", "invariants", "caveats" |

### Plain-text files (.cursorrules)

These are typically terse and instruction-shaped. Required content is content-based, not section-based:

- At least one statement about the project's domain or stack.
- At least one explicit instruction or constraint.

No section headers required. Length budget is tighter (see below).

---

## Length budget

| File | Soft cap (warn) | Hard cap (critical) | Minimum (warn) |
|------|-----------------|---------------------|----------------|
| CLAUDE.md | 250 lines | 400 lines | 30 lines |
| AGENTS.md | 250 lines | 400 lines | 30 lines |
| .github/copilot-instructions.md | 200 lines | 350 lines | 20 lines |
| .cursorrules | 80 lines | 150 lines | 5 lines |

Soft cap: emit `lint:length` finding. Hard cap: emit `lint:length-critical`. Minimum: emit `lint:thin` (likely under-documented).

Length counts non-blank, non-frontmatter lines. Code blocks count.

---

## Anti-pattern detectors

Each detector below produces a `lint:anti-pattern` finding when matched. **Advisory only** — surfaced in the report, not auto-removed.

### AP-1: Directory catalogue

Lists that restate the file tree without adding insight.

**Detect:** any list (markdown bullet or numbered) where ≥4 consecutive items match the pattern:
```
- `<path>/` — <noun phrase that contains "code", "files", "logic", "stuff", or repeats the path>
```

Example to flag:
```
- `components/` — contains components
- `utils/` — utility code
- `hooks/` — hook files
- `pages/` — page components
```

Example NOT to flag (these add insight):
```
- `internal/auth/` — JWT signing lives here, not in `pkg/`; tokens use ed25519 not RSA
- `cmd/migrator/` — out-of-process migration runner, called from CI only
```

### AP-2: Human onboarding fluff

Content addressed to a human reader, not an AI agent. The audience for these files is the AI.

**Detect:** any of these phrases (case-insensitive), each match is one finding:
- "welcome to"
- "thank you for"
- "we hope you"
- "feel free to"
- "happy coding"
- "🎉" or "🚀" or other decorative emoji in headings or summary text
- "mission", "vision", "values" as section headings

### AP-3: Style rules inferable from code

Style/naming rules that a competent AI would learn from looking at any source file.

**Detect:** lines matching these patterns when the convention is already visible in ≥80% of repo source files:
- "use camelCase" / "use snake_case" / "use PascalCase"
- "indent with N spaces" / "use tabs"
- "files end with newline"
- "use single quotes" / "use double quotes"

Verify the inferability by sampling 10 source files of the repo's primary language and checking if the rule is followed. If yes → flag as redundant. If the repo is inconsistent → don't flag (the rule has teeth).

### AP-4: Manifest duplication

Content that just restates what's in `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `composer.json`, `Gemfile`, etc.

**Detect:**
- Full dependency lists ("Dependencies: react@19, vite@6, …")
- Restated script tables when the file says "run `npm test` to test, `npm run build` to build, …" and those scripts are literally in `package.json` with no extra context.

The AI reads manifests directly. Only document commands when there's **extra context** the AI can't derive (e.g. "use `make test-integration` not `go test ./...` — the latter doesn't spin up Postgres").

### AP-5: Stale "coming soon" / TODO

**Detect:**
- Lines containing "TODO", "FIXME", "coming soon", "WIP", "work in progress", "to be added", "planned"
- Sections titled "Future plans", "Roadmap", "Upcoming features"

If the same TODO appeared in the file 90+ days ago (check `git log -L`), it's almost certainly abandoned content. Flag with the age.

### AP-6: Marketing language

**Detect:** the file is documentation for an AI, not a landing page. Flag phrases like:
- "blazing fast", "industry-leading", "best-in-class"
- "comprehensive", "robust", "powerful" when applied to the project itself
- Badge clusters (more than 2 consecutive shields.io / badge lines)
- Logo references at the top

These belong in `README.md`, not in the AI-context file.

### AP-7: Repeated content

Multiple sections that say the same thing in different words. **Detect:** two paragraphs ≥3 sentences each with ≥60% n-gram overlap. Flag the second occurrence.

---

## Healthy file shape (reference)

A healthy CLAUDE.md for a medium codebase typically looks like:

```
# CLAUDE.md

<1–3 sentence overview>

## Architecture

<5–30 lines covering non-obvious structure, key relationships, where the
"real" logic lives vs. boilerplate>

## Commands

<5–15 lines of project-specific commands with extra context — not a
restatement of package.json scripts>

## Conventions

<5–25 lines of project-specific norms: commit format, file placement
rules, naming patterns, framework choices that matter>

## Gotchas / What Not to Do

<3–20 lines of high-value land mines: "don't X because Y", invariants
that aren't obvious from the code>
```

Total: 80–200 lines. If yours is materially smaller, you're probably under-documented. If materially larger, you're probably accreting bloat.

---

## What this spec deliberately does NOT enforce

- **Section ordering.** Whatever order makes sense for the project is fine.
- **Heading levels.** `#` vs `##` is the author's choice.
- **Tone.** Terse, conversational, or formal — all acceptable.
- **Markdown extensions.** Tables, callouts, etc. are fine if they aid comprehension.
- **Inclusion of optional sections** (testing strategy, deployment, CI overview, etc.). Add them if they're high-signal for your repo.

The spec enforces **truth and signal density**, not stylistic uniformity.
