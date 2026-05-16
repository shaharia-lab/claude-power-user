# File Format Conventions

Per-file quirks for the four AI-context file types `agent-context-sync` handles. Used during discovery, verification, lint, and edit phases.

> **Principle:** Each file has its own audience, convention, and edit rules. Treating them identically produces sloppy output. Treating them as wildly different produces complexity. This file documents the minimum differentiation needed.

---

## CLAUDE.md

**Audience:** Claude Code (Anthropic's CLI). Also generally readable by other AI tools.

**Format:** Markdown. CommonMark.

**Typical location:** repo root.

**Conventions:**
- Top-level heading is `# CLAUDE.md`.
- Section headings use `##`.
- Code fences use triple backticks with language hints.
- Tables are fine and often useful for plugin/agent inventories.
- YAML frontmatter is rare (don't add one).

**Edit rules:**
- Preserve existing heading structure when fixing claims.
- When adding a new section in Phase 4 (augmentation), use `##` and place it after the section it logically extends, or at the end if no obvious home exists.
- Preserve blank line conventions (one blank line between top-level sections is standard).

**Length budget:** soft 250, hard 400, min 30.

---

## AGENTS.md

**Audience:** Multi-tool — emerging convention for AI coding assistants. Often referenced by Cursor, Continue, and similar tools.

**Format:** Markdown. CommonMark.

**Typical location:** repo root.

**Conventions:**
- Top-level heading is `# AGENTS.md` or `# <Project> Agents`.
- Tends to be terser than CLAUDE.md (more imperative, less explanatory).
- Often heavy on bullet lists of "rules" and "constraints".
- Less narrative than CLAUDE.md.

**Edit rules:**
- Match the existing terseness when adding content.
- If CLAUDE.md has a long architectural narrative and AGENTS.md doesn't, do **not** copy the narrative over — extract the imperative rules from it instead.
- New sections use the same heading level as existing ones (usually `##`).

**Length budget:** soft 250, hard 400, min 30.

**Note on duplication with CLAUDE.md:** if both files exist in a repo, they often duplicate content. This is acceptable — they serve different tools. Cross-file divergence findings should flag **factual** disagreements (versions, paths, counts), not structural or stylistic ones.

---

## .cursorrules

**Audience:** Cursor IDE's AI assistant.

**Format:** Plain text. Not Markdown. No headings, no code fences, no formatting.

**Typical location:** repo root.

**Conventions:**
- Short. Often 10–80 lines.
- Imperative voice ("Use X. Prefer Y. Never Z.").
- One instruction per line is typical.
- Sometimes structured as a numbered list, sometimes as plain paragraphs.
- No frontmatter, no headings, no decorative formatting.

**Edit rules:**
- **Never add Markdown formatting.** No headings, no `**bold**`, no fenced code blocks. Cursor may render or strip these unpredictably.
- Preserve line structure: if the file uses one-instruction-per-line, keep that. If it uses paragraphs, keep paragraphs.
- New rules are appended as single lines at the bottom, preceded by a blank line if the existing file uses blank-line separators.
- Avoid prose. Cursor users expect terse directives.

**Length budget:** soft 80, hard 150, min 5.

**Section-presence check:** `.cursorrules` does NOT use section headings. The Phase 3 "required sections" check is replaced with content-based checks:
- At least one statement mentions the project's domain or stack.
- At least one explicit directive (must/should/never/always/prefer/avoid).

---

## .github/copilot-instructions.md

**Audience:** GitHub Copilot's coding agent (the workspace-level instructions file).

**Format:** Markdown. CommonMark.

**Typical location:** `.github/copilot-instructions.md` exactly. Not elsewhere.

**Conventions:**
- Top-level heading is `# Copilot Instructions` or `# Workspace Instructions`.
- Often shorter than CLAUDE.md (Copilot's context window for these is more constrained).
- Heavy on coding conventions, less on architecture.
- Often references the language/framework prominently.

**Edit rules:**
- Same Markdown conventions as CLAUDE.md.
- Be especially aggressive about brevity — Copilot truncates long instruction files.
- Don't duplicate the entire CLAUDE.md here; this file is for Copilot-specific guidance plus the essential coding rules.

**Length budget:** soft 200, hard 350, min 20.

---

## Cross-file canonical selection

When multiple files exist and `--canonical` is not specified:

1. If `CLAUDE.md` exists → it is canonical.
2. Else if `AGENTS.md` exists → it is canonical.
3. Else if `.github/copilot-instructions.md` exists → it is canonical.
4. Else `.cursorrules` is canonical by default.

Canonical only matters for:
- Cross-file divergence reporting (the canonical value is shown as the suggested resolution).
- The `--since` default in augmentation (defaults to the canonical file's last-modifying commit).

It does **not** mean the canonical file's content is mirrored into others. Mirroring is out of scope for v1.

---

## File-not-found behavior

| File | If missing |
|------|------------|
| CLAUDE.md | Recommend creation in dry-run report; do not auto-create. |
| AGENTS.md | Silent (don't recommend creation; not all projects need it). |
| .cursorrules | Silent. |
| .github/copilot-instructions.md | Silent. |

The skill is a **doctor**, not a midwife. It treats existing files; it doesn't bootstrap missing ones. (A future `agent-context-bootstrap` skill could fill that gap.)

---

## Whitespace and EOL conventions

- Preserve the file's existing line endings (LF vs CRLF). Don't normalize.
- Preserve trailing newlines: if the file ends with a newline, keep it; if not, don't add one.
- Indentation in nested lists: match the file's existing convention (2 spaces, 4 spaces, or tabs — sample the first nested list to detect).
- Never reformat untouched content. Edits should be local.
