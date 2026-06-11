---
name: review-rule
description: Review rule documentation pages against the site's authoring conventions. Use for "review this rule page", "check conventions", or as a pre-PR check on changed pages. Reports findings; fixes only on request or with --fix.
argument-hint: "[rule-id | path] [--fix]"
---

# Review rule pages

Review one or more rule pages against `.claude/rules/rule-pages.md`. Read that file first — it is the checklist; this skill only defines targeting, reporting, and fixing.

## Step 1 — Resolve targets

From `$ARGUMENTS` (the `--fix` flag may appear alongside a target):

- A rule ID (`^[A-Z]{2}[0-9]{4}$`, any case) → that rule's page in the analyzer folder matching the prefix (AC → ApplicationCop, DC → DocumentationCop, FC → FormattingCop, LC → LinterCop, PC → PlatformCop, TA → TestAutomationCop). Page bundles (`<RULEID>/index.md`) count.
- A file path → that page.
- No target → all uncommitted rule pages: parse `git status --porcelain` for modified, staged, and untracked files under `content/docs/analyzers/` (ignore `_index.md` as a target, but see the index check below).

If this resolves to nothing, say so and stop.

## Step 2 — Review each page

Check the page against every applicable convention in `.claude/rules/rule-pages.md`. In particular:

**Front matter**
- TOML (`+++`); `title` ends with ` (XX0000)`; `linkTitle` is the bare ID.
- `[params]` present with valid values (severity, category from the allowed lists; `codeActionType`/`supportsFixAll` only meaningful when `codeAction = true`). A legacy page without `[params]` gets a *suggestion*-level finding, not a must-fix — unless the page is new in this changeset, then it's a must-fix.
- No manually written properties table in the body (the partial renders it).

**Structure**
- Body sections present and in the canonical order; optional sections only where they make sense.
- Why section opens the body without a heading and follows the three beats (mechanism → concrete consequence → fix sentence); doesn't restate the title.
- Bad example carries the inline diagnostic comment `// <Title> [XX0000]`; the fixed example doesn't.

**Style**
- No transition filler, hedging, vague warnings ("may lead to unexpected behavior"), rule-selling, or generic tips sections.
- Short declarative sentences; mechanism before imperative.
- Realistic AL objects in examples where the concept allows it.

**Hugo**
- Internal links lowercase (`[PC0034](pc0034/)`).
- Shortcodes well-formed; images referenced via `imgproc` from a page bundle.

**Index consistency**
- The analyzer's `_index.md` has a row for the rule, lowercase-linked, with Title/Severity/Code Fix matching the page, sorted by ID.

**Metadata cross-check (when `../Analyzers` exists)**
- Verify `[params]` values against the analyzer source: rule name via `DiagnosticIds.cs`, severity/category via `DiagnosticDescriptors.cs`, code fix existence/`Kind`/`SupportsFixAll` via `CodeFixes/<RuleName>.cs`, obsolete handling via `IsObsolete()` usage in `Analyzers/<RuleName>.cs`. Flag any drift as must-fix. Skip silently (with a note in the report) when the sibling repo isn't available.

## Step 3 — Report

Group findings per page. For each finding give `file:line`, a severity — **must-fix** (violates the conventions or contradicts the analyzer source) or **suggestion** (style improvement, legacy `[params]` gap) — and the concrete proposed correction, quoting replacement text where short.

If everything passes, say so explicitly per page.

## Step 4 — Fix (only when asked)

Apply corrections **only** when `--fix` was passed or the user asks after seeing the report. Then:

- Apply must-fix findings; apply suggestions unless they would rewrite substantial prose the author may want to keep — list anything you deliberately left.
- Re-run `hugo --quiet` from the repo root and surface any errors.
- Summarize what changed per file.

Without `--fix`, end at the report. Do not edit files.
