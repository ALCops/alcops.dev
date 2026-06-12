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
- `[params]` present with valid values (severity, category from the allowed lists; `codeAction` is a plain bool). A legacy page without `[params]` gets a *suggestion*-level finding, not a must-fix — unless the page is new in this changeset, then it's a must-fix.
- No manually written properties table in the body (the partial renders it).

**Structure**
- First determine the page tier (see "Body structure" in the conventions): standard pages must have the canonical section order; concept-heavy pages (metric/methodology/platform-machinery rules with a free-form outline) are reviewed only against the reduced mandatory set — front matter `[params]`, opening Why without a heading, at least one bad/fixed example pair. Do **not** flag extra sections, custom headings, or named nuance subsections on a concept-heavy page as violations.
- Why section opens the body without a heading and follows the three beats (developer intent → one concrete named failure → fix sentence); doesn't restate the title.
- Bad example carries the inline diagnostic comment `// <Title> [XX0000]`; the fixed example doesn't.
- Both code examples use the Hugo highlight shortcode with `hl_lines`: the bad example highlights where the diagnostic is raised, the fixed example highlights the changed lines.
- No `### When the diagnostic is NOT reported` section — this is a legacy pattern (applies to both tiers). If found, flag as **must-fix**: remove it and move any surprising exclusions to `### Exception`.

**Style**
- Categorical consequence lists (chains of abstract harm categories like "data integrity issues, constraints not enforced, subscribers never fire" in place of one concrete named failure) → **must-fix**.
- A textbook opening that states the mechanism in a vacuum without connecting to what the developer was trying to do → **suggestion**: rework per the Voice exemplars in the conventions.
- No separate rendered diagnostic message block after the bad example — the inline comment is sufficient. Flag any such block as **must-fix**: remove it.
- No transition filler, hedging, vague warnings ("may lead to unexpected behavior"), rule-selling, or generic tips sections.
- Short declarative sentences; mechanism before imperative.
- Realistic AL objects in examples where the concept allows it.
- Inline links and `>` blockquotes are valid per the conventions — do not flag them. Do check that every blockquote carries an attribution link and that no URL appears both inline and in `### See also`.

**Hugo**
- Internal links lowercase (`[PC0034](pc0034/)`).
- Shortcodes well-formed; images referenced via `imgproc` from a page bundle.

**Index consistency**
- The analyzer's `_index.md` has a row for the rule, lowercase-linked, with Title/Severity/Code Fix matching the page, sorted by ID.

**Metadata cross-check (when `../Analyzers` exists)**
- Verify `[params]` values against the analyzer source: rule name via `DiagnosticIds.cs`, severity/category via `DiagnosticDescriptors.cs`, code fix existence via `CodeFixes/<RuleName>.cs`, obsolete handling via `IsObsolete()` usage in `Analyzers/<RuleName>.cs`. Flag any drift as must-fix. Skip silently (with a note in the report) when the sibling repo isn't available.

**See also enrichment**
- Skip this check for rules about simple naming, formatting, or style conventions where the page title alone conveys the full concept.
- For rules involving platform behavior, performance, SQL generation, permissions, events, or complex concepts: evaluate whether the page already has a `### See also` section with relevant external links.
- If the page lacks a See also section (or has only internal cross-references) and the concept has depth, search for candidate links:
  1. Use `microsoft_docs_search` with the underlying concept and "Business Central" (e.g., "SetLoadFields partial records Business Central", not the rule ID). Include the AL property or method name when one exists.
  2. Use `WebSearch` for community content: search for the concept plus "Business Central", optionally scoping to known sources (vjeko.com, erikhougaard.com, demiliani.com, yzhums.com, waldo.be, kauffmann.nl).
  3. For each candidate URL, use `WebFetch` (or `microsoft_docs_fetch` for learn.microsoft.com URLs) to confirm the page exists and is relevant to the rule's concept — do not suggest links you haven't verified.
- Report found links as a **suggestion**-level finding: "Consider adding a See also section with:" followed by a formatted bullet list using the site's link conventions (`[Title](URL) on Microsoft Learn`, `[Title](URL) by Author Name`). When a source pins down the precise behavior the page paraphrases weakly, suggest quoting it inline as a `>` blockquote with attribution instead of (not in addition to) listing it.
- If the page already has a See also section, verify that existing URLs are not broken (fetch each to confirm). Report dead links as **must-fix**.

## Step 3 — Report

Group findings per page. For each finding give `file:line`, a severity — **must-fix** (violates the conventions or contradicts the analyzer source) or **suggestion** (style improvement, legacy `[params]` gap) — and the concrete proposed correction, quoting replacement text where short.

If everything passes, say so explicitly per page.

## Step 4 — Fix (only when asked)

Apply corrections **only** when `--fix` was passed or the user asks after seeing the report. Then:

- Apply must-fix findings; apply suggestions unless they would rewrite substantial prose the author may want to keep — list anything you deliberately left.
- Re-run `hugo --quiet` from the repo root and surface any errors.
- Summarize what changed per file.

Without `--fix`, end at the report. Do not edit files.
