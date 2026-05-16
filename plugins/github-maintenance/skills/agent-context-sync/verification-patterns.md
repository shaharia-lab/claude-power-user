# Verification Patterns

Concrete patterns for extracting verifiable claims from AI-context files, and the checks to run against each. Used by `agent-context-sync` in Phase 2.

> **Principle:** Only verify claims that have a deterministic check. If a claim's truth depends on runtime behavior, semantic interpretation, or domain knowledge, mark it `unverifiable` and leave it alone.

Each pattern below has four parts: **regex**, **extraction**, **check**, **fix-derivation**.

---

## VP-1: Counts of well-known repo entities

**Regex:** `\b(\d+)\s+(agents?|skills?|plugins?|modules?|services?|packages?|commands?|hooks?|endpoints?|migrations?|tables?)\b`

**Extraction:** Match number + entity type. Example matches:
- "20 agents"
- "9 composable skills"
- "3 plugins"

**Check:** Count actual occurrences in the repo using entity-specific rules:

| Entity | How to count |
|--------|--------------|
| agents | `find . -path '*/agents/**/*.md' \| wc -l` (or plugin-specific path) |
| skills | Count directories containing `SKILL.md` |
| plugins | Count entries in `.claude-plugin/marketplace.json` plugins array, OR count subdirs of `plugins/` with a `plugin.json` |
| modules / packages | Count `package.json` / `go.mod` / `Cargo.toml` / `pyproject.toml` files OR exported modules |
| services | Top-level dirs under `services/` or `cmd/` |
| commands | Files under `commands/` or scripts in `package.json` |
| migrations | Files in `migrations/` |
| endpoints | Lines matching route registration patterns (framework-specific; only count if unambiguous) |
| tables | Migration files defining `CREATE TABLE` (if SQL repo) |

**Fix derivation:** If the claim says "N X" and the count is M ≠ N, propose replacement with "M X" verbatim. Only auto-fix when the count is unambiguous.

**Tolerance:** If the count is within ±1 (off-by-one due to plurals or in-progress files), emit `wrong` only if there's clear evidence of a planned count (e.g. matches a version-bumped file). Otherwise emit `stale` so the human can decide.

---

## VP-2: Version strings

**Regex:** `\b(v?\d+\.\d+(?:\.\d+)?(?:-[a-z0-9.-]+)?)\b` near a known label

**Extraction:** Capture the version AND the label preceding it within ~5 words. Labels to match:
- A tool name: "Go 1.22", "Node 18", "React 19", "Python 3.11", "Bun 1.0"
- A plugin/package name from the repo
- The phrase "version" or "v"

**Check:** Compare against the source of truth for that label:

| Label source | Where to read truth |
|--------------|---------------------|
| Go | `go.mod` `go` directive |
| Node | `package.json` `engines.node` or `.nvmrc` |
| Python | `pyproject.toml` `requires-python` or `.python-version` |
| Rust | `Cargo.toml` `rust-version` |
| A package from `package.json` | `dependencies.<name>` or `devDependencies.<name>` |
| A repo-local plugin | The plugin's `plugin.json` `version` |
| The marketplace itself | `.claude-plugin/marketplace.json` `metadata.version` |

**Fix derivation:** If the documented version differs from the source-of-truth version, propose replacement with the source-of-truth value. Use major-only, major.minor, or major.minor.patch matching the original claim's precision.

**Skip:** if the source of truth is itself missing (no `go.mod`, no `package.json`), the claim is `unverifiable`, not `wrong`.

---

## VP-3: Backticked paths

**Regex:** Backtick-delimited tokens containing `/` or ending in a file extension. `\`([^\`]*(?:/[^\`]*|\.\w+))\``

**Extraction:** Capture the path. Strip leading/trailing slashes for the existence check but preserve them for the fix.

**Check:**
- Resolve relative to the repo root.
- Run `test -e <path>` semantics. Glob patterns (`*`, `?`, `[...]`) are expanded; the claim passes if at least one match exists.
- Symlinks are followed.

**Fix derivation:** If the path doesn't exist, attempt to find a near-match:
- Same basename elsewhere in the repo (`find . -name '<basename>'`). If exactly one match, propose the corrected path.
- Same directory name in a moved location. If exactly one match, propose it.
- Otherwise: mark `stale` with no fix.

**Skip:** paths that look like:
- URLs (contains `://`)
- Code references with line numbers (`file.ts:42`)
- Placeholder paths (`<your-org>/...`, `path/to/your/file`, `path/to/...`, anything with angle brackets or "TODO")
- Paths outside the repo (absolute paths not starting with the repo root)

---

## VP-4: Build/test/lint commands

**Regex:** Code fences or backticked single-line commands starting with known tool invocations:
- `npm run <script>` / `yarn <script>` / `pnpm <script>` / `bun <script>`
- `make <target>`
- `cargo <subcommand>`
- `go <subcommand>`
- `python -m <module>` / `pytest`
- `mvn <goal>` / `gradle <task>`
- `<repo-script>` matching an executable file in the repo

**Extraction:** Capture the runner and the script/target name.

**Check:**

| Runner | Source of truth |
|--------|-----------------|
| `npm run X` / `yarn X` / `pnpm X` / `bun X` | `package.json` `scripts.X` exists |
| `make X` | `Makefile` defines target `X:` |
| `cargo X` | `X` is a built-in cargo subcommand OR a `Cargo.toml` alias |
| Custom script paths | File exists and is executable |

**Fix derivation:**
- If the script/target doesn't exist but a similar one does (Levenshtein ≤2), propose the closest match.
- Otherwise mark `stale` with no fix.

**Skip:** common invocations that are stable across all projects of that language (`go test ./...`, `npm install`, `cargo build`) — these don't need verification.

---

## VP-5: "Lives at" / "located in" / "under" path claims

**Regex:** `(lives? (?:at|in)|located (?:at|in)|found (?:at|in)|under)\s+\`([^\`]+)\``

**Extraction:** Capture the path inside the backticks.

**Check:** Same as VP-3, but additionally verify the path is non-empty (the directory or file contains content). An empty directory is `stale`.

**Fix derivation:** Same as VP-3.

---

## VP-6: Section/heading references

**Regex:** Phrases like "see the X section" or "(see [Section Name](#anchor))" or "described in §X"

**Extraction:** Capture the referenced section name.

**Check:** Verify the referenced section exists in the same file (or in the linked file).

**Fix derivation:** If the section was renamed, propose the closest match by token overlap. Otherwise mark `stale`.

---

## VP-7: Stack/framework declarations

**Regex:** Statements asserting the repo uses a specific framework or tool, e.g.:
- "uses React" / "built with Vue"
- "PostgreSQL 15"
- "Chi router"
- "Wire DI"

**Extraction:** Capture the framework/tool name.

**Check:** Search the repo's manifest files (`package.json`, `go.mod`, etc.) and imports for evidence of the framework. The framework must appear as a direct dependency or in import statements.

**Fix derivation:** If the manifest no longer contains the framework, mark `stale` with the actual stack (top dependencies) as evidence. Do **not** auto-fix — stack claims often have nuance ("legacy code still uses X" etc.).

---

## VP-8: Cross-file divergence

After all per-file verification is complete, build a fact index: `{label → {file → value}}`. For each label appearing in 2+ files with disagreeing values, emit a `divergence` finding.

**Examples:**
- CLAUDE.md: "Go 1.22"; `.cursorrules`: "Go 1.20" → divergence on Go version
- CLAUDE.md: "20 agents"; AGENTS.md: "19 agents" → divergence on agent count

**Resolution:** The canonical file's value is shown as the suggested value. Findings are advisory — not auto-applied even in PR mode.

---

## Patterns deliberately NOT included

These exist as ideas but are excluded due to high false-positive rates:

- **English assertions about behavior** ("the cache expires every 5 minutes") — unverifiable without runtime testing.
- **Architectural claims** ("clean architecture") — semantic, not deterministic.
- **Performance numbers** ("87–92% token savings") — claims about external benchmarks.
- **Examples in code blocks** — illustrative, not normative.

If a future iteration of this skill adds runtime checks, these can move into the verifiable set.

---

## Implementation guidance for the skill

When extracting claims:

1. Process the file once, line by line, recording line numbers.
2. Apply patterns in the order above. A line can match multiple patterns; collect all matches.
3. Skip content inside fenced code blocks for VP-1 and VP-7 (they're examples, not assertions). VP-3, VP-4, VP-5 still apply inside code blocks.
4. Skip the frontmatter block at the top of markdown files.
5. Deduplicate: if the same path appears in 5 places, run the check once and apply the result to all 5 occurrences.

When running checks:

1. Batch by check type — count all entities once per type, read each manifest once.
2. Cache results within a single skill invocation.
3. Use `git ls-files` (not `find`) for path existence so untracked files don't pollute results.
