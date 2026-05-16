---
name: agent-context-sync
description: Periodic, repo-agnostic doctor for AI-context files (CLAUDE.md, AGENTS.md, .cursorrules, .github/copilot-instructions.md). Verifies every concrete claim against the current repo, lints against an opinionated quality spec, surfaces newly-added subsystems the doc misses, and proposes the minimum edits to restore health. Conservative pruning. No-op when files are already healthy — safe to run on a cron.
argument-hint: "[--dry-run] [--files=<comma-separated-paths>] [--canonical=<path>] [--since=<git-revision>]"
disable-model-invocation: true
---

# AGENT CONTEXT SYNC

Keep a repository's AI-context files accurate, lean, and useful. This skill is **user-triggered only** — never auto-invoke it. It edits documentation (or opens a PR), which the user must explicitly approve via the slash command.

---

## INVARIANTS

These rules apply to every run. Do not restate them, just follow.

1. **Conservative pruning.** Only delete claims you can programmatically prove are wrong or stale (failed path check, mismatched version, miscount, missing command/script). Anything else — bloat, suspected anti-patterns, unverifiable claims — is **flagged in the report but left in place**. Humans decide.
2. **No-op when healthy.** If every claim verifies and lint passes and the recent git delta surfaces no missing content, print the healthy banner and exit. Do not create a branch. Do not open a PR. Do not touch files.
3. **One pass, deterministic.** Same inputs → same output. No "creative rewriting" — every edit must trace back to a finding in the verification or lint phase.
4. **Repo-agnostic.** No hardcoded paths, languages, or frameworks. Detect everything from the repo itself.
5. **Preserve voice and curated content.** If the existing file has a "What Not to Do" section or specific gotchas, keep them. The goal is correctness, not stylistic homogenization.
6. **Never invent.** If you can't verify a claim, leave it and flag it as `unverifiable` in the report. Don't replace it with a guess.
7. **Reference files are authoritative.** Read [`quality-spec.md`](quality-spec.md), [`verification-patterns.md`](verification-patterns.md), and [`file-format-conventions.md`](file-format-conventions.md) before phase 2.

**Logging format (mandatory):**
```
[ACS-Pn] [ACTION|RESULT|DECISION|COMPLETE] message
```
where `n` is the phase number.

---

## ARGUMENTS

- `--dry-run` — print the structured report to stdout. No file edits, no branch, no PR. Default is PR mode.
- `--files=<paths>` — comma-separated list of AI-context files to check. Overrides discovery. Useful for non-standard layouts.
- `--canonical=<path>` — designate one file as the canonical source for cross-file divergence comparisons. Defaults to CLAUDE.md if present, else the most recently modified file.
- `--since=<git-revision>` — git revision to diff against for the augmentation phase. Defaults to the canonical file's last-modifying commit.

---

## PHASES

Run in order. Do not skip. Do not reorder.

### PHASE 0: Pre-flight

1. Confirm you are inside a git repository: `git rev-parse --show-toplevel`. If not, print an error and stop.
2. Confirm the working tree is clean (`git status --porcelain`). If dirty and **not** `--dry-run`, refuse: tell the user to stash or commit first. Dirty trees are allowed in `--dry-run`.
3. Resolve the current branch name. If on the repository's default branch and **not** `--dry-run`, you'll be creating a new branch in Phase 5 — that's fine.
4. Read the three reference files in this skill directory: `quality-spec.md`, `verification-patterns.md`, `file-format-conventions.md`.

Log: `[ACS-P0] [COMPLETE] preflight ok, repo=<name>, branch=<current>, dry-run=<bool>`

### PHASE 1: Discover

Find AI-context files. Check in this exact order, recording which exist and their last-modified commit (`git log -1 --format=%H:%ct -- <path>`):

| Path | Format |
|------|--------|
| `CLAUDE.md` | Markdown |
| `AGENTS.md` | Markdown |
| `.cursorrules` | Plain text |
| `.github/copilot-instructions.md` | Markdown |

Also check if `--files` overrides this list. If yes, only operate on those.

**Canonical selection:**
- If `--canonical` provided, use it.
- Else if `CLAUDE.md` exists, it's canonical.
- Else use the file with the most recent last-modifying commit.

If zero AI-context files exist:
- In `--dry-run`: print a report recommending creation of `CLAUDE.md` based on detected stack, with a suggested skeleton. Exit.
- In PR mode: refuse with `[ACS-P1] [DECISION] no AI-context files found — bootstrapping is out of scope for this skill. Create a starter CLAUDE.md manually first, then re-run.` Exit.

Log each discovered file: `[ACS-P1] [RESULT] found=<path> last_modified=<iso8601> bytes=<n> lines=<n>`

### PHASE 2: Verify claims

For each discovered file, extract verifiable claims using the patterns in `verification-patterns.md` and run each check. Build a per-file finding list. Each finding has:

```
{
  file: <path>,
  line: <n>,
  claim: <verbatim quote>,
  pattern: <which extraction pattern matched>,
  check: <what was tested>,
  status: ok | wrong | stale | unverifiable,
  evidence: <command output or file contents proving the status>,
  proposed_fix: <if status is wrong/stale, the corrected text — else null>
}
```

**Status definitions:**
- `ok` — check passed, claim is current.
- `wrong` — check failed and there is a clearly correct replacement (e.g. file says "version 1.0.4" but `plugin.json` says "2.0.0").
- `stale` — check failed and there is no clear replacement (e.g. claim references a deleted directory). Conservative pruning means: flag, but only delete if also referenced by another `wrong` finding that supersedes it.
- `unverifiable` — pattern matched but the check can't be run automatically (e.g. claim about runtime behavior, "the cache invalidates every 5 minutes"). Always left in place.

Do not produce findings for content that doesn't match any extraction pattern. Lint handles unstructured concerns separately.

**Cross-file divergence pass:** for each verifiable fact appearing in 2+ files (e.g. a version number, a build command), if the files disagree, emit a `divergence` finding scoped to the non-canonical files with the canonical value as `proposed_fix`. Note: divergence findings are advisory — they are reported but NOT auto-fixed even in PR mode, because the non-canonical file might intentionally lag (e.g. `.cursorrules` is rebuilt by a separate tool).

Log a summary per file: `[ACS-P2] [RESULT] file=<path> ok=<n> wrong=<n> stale=<n> unverifiable=<n>`

### PHASE 3: Lint against quality spec

For each file, run the checks in `quality-spec.md`:

- **Required sections present?** Use the section list appropriate to the file format (markdown vs plain-text). Missing sections produce `lint:missing-section` findings.
- **Length within budget?** Soft cap, hard cap, and minimum from `quality-spec.md`. Produce `lint:length` finding if outside soft cap; `lint:length-critical` if outside hard cap.
- **Anti-pattern scan.** Run each detector in `quality-spec.md` (directory catalogue, fluff, redundant style rules, manifest duplication, stale TODOs). Produce `lint:anti-pattern` findings with the matched lines.

Anti-pattern findings are **advisory only** under the conservative pruning policy. They surface in the report but are not auto-removed.

Log: `[ACS-P3] [RESULT] file=<path> lint_findings=<n> antipatterns=<n>`

### PHASE 4: Augment

Look for material additions to the repository that the canonical file does not mention.

1. Determine the diff base: `--since` argument, or the canonical file's last-modifying commit (`git log -1 --format=%H -- <canonical>`).
2. Compute `git log --since-as-of=<sha> --name-status` to get changed paths. Group by top-level directory.
3. For each significantly-changed area (new top-level directory, new manifest file, new build script, ≥10 changed files in a subsystem), check whether the canonical file mentions it. If not, emit a `missing-coverage` finding with:
   - The area (e.g. "new plugin `plugins/github-maintenance/`")
   - Evidence (commit list, file count)
   - Suggested addition (1–3 sentence summary, written in the file's existing voice)

**Filter:** do NOT propose additions for content an AI could discover by grep in 30 seconds. Examples to skip: a new test file, a renamed variable, a CSS tweak, a single new endpoint within an already-documented subsystem.

Log: `[ACS-P4] [RESULT] missing_coverage_findings=<n>`

### PHASE 5: Output

#### If `--dry-run`:

Print a single Markdown report to stdout with this structure:

```markdown
# agent-context-sync report

**Repo:** <owner>/<name>
**Canonical file:** <path>
**Run mode:** dry-run
**Result:** <healthy | findings>

## Summary

| File | OK | Wrong | Stale | Lint | Missing coverage |
|------|----|----|------|------|------------------|
| ... | ... | ... | ... | ... | ... |

## Findings

### <file>

#### Wrong (will fix in PR mode)
- **Line N:** `<claim>` — <reason>. Proposed: `<fix>`.

#### Stale (will fix in PR mode)
- ...

#### Lint (advisory, not auto-fixed)
- ...

#### Missing coverage (will add in PR mode)
- ...

#### Cross-file divergence (advisory, not auto-fixed)
- ...

## Recommended actions

- <bulleted list, prioritized>
```

If there are zero findings of any kind, print only:
```
[agent-context-sync] healthy — no changes proposed.
```

Exit.

#### If PR mode and zero findings:

Print the healthy banner (same as above). Do not create a branch. Do not open a PR. Exit.

#### If PR mode and findings exist:

1. Create branch: `claude/agent-context-sync-<YYYYMMDD-HHMM>` from the current branch.
2. For each `wrong` and `stale` finding with a `proposed_fix`, apply the edit. Use exact-string replacement; preserve surrounding context including indentation and whitespace.
3. For each `missing-coverage` finding, insert the suggested addition into the most appropriate existing section. If no section fits, append a new section in the file's existing style.
4. Do NOT apply lint findings or divergence findings (conservative policy).
5. Stage and commit with message:
   ```
   docs: refresh AI-context files (agent-context-sync)

   - <count> wrong claims corrected
   - <count> stale references removed
   - <count> new subsystems documented

   Lint findings and cross-file divergences reported but not auto-fixed (conservative policy).
   ```
6. Push the branch.
7. Open a PR against the repo's default branch with:
   - Title: `docs: refresh AI-context files (agent-context-sync)`
   - Body: the same Markdown report from the dry-run output, prefixed with a one-paragraph summary.
8. Print the PR URL and exit.

Log: `[ACS-P5] [COMPLETE] mode=<dry-run|pr> findings_applied=<n> pr_url=<url-or-null>`

---

## OUTPUT CONTRACT

**Always print exactly one report or one healthy banner per run.** Never print partial progress to stdout — use the `[ACS-Pn]` log lines for that, on stderr if your environment separates streams.

The report is the deliverable. The PR (when in PR mode) is a delivery mechanism for the same report plus the corresponding edits. A human reading the PR must be able to understand every change by reading only the report.

---

## FAILURE MODES TO AVOID

- **Rewriting the whole file.** You are a doctor, not a ghostwriter. If you find yourself touching more than 20% of the file in a single run, stop and reconsider — you are probably regenerating instead of repairing.
- **Generating prose to fill perceived gaps.** Missing-coverage findings should be terse and trace to specific git history. If you can't point to a commit or directory, don't add the content.
- **Touching files outside the AI-context set.** This skill edits only the files discovered in Phase 1. Never edit code, manifests, or other docs.
- **Auto-fixing lint or divergence findings.** These are advisory. Surface them; let humans decide.
- **Running without a clean working tree in PR mode.** Refuse, don't merge your edits with the user's in-flight work.
