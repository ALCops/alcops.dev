# Contributing to ALCops Documentation

Thank you for considering a contribution. This site is the reference documentation for the [ALCops](https://github.com/ALCops) code analyzers. Writing documentation isn't glamorous, but it's what turns "works on my machine" into something everyone can actually use. If you're here, you’re already doing the kind of work most of us quietly postpone.

Pull requests are the preferred way to contribute. No contribution is too small. A typo fix or a clarified sentence is just as welcome as a completely new page.

## What you can contribute

- Fix typos, grammar, or unclear wording
- Improve or extend examples for existing analyzer rules
- Add missing documentation for a rule
- Improve getting started or configuration guides
- Fix broken links or outdated information

## How to contribute

1. **Fork** the repository on GitHub: [ALCops/alcops.dev](https://github.com/ALCops/alcops.dev)

2. **Clone** your fork and set up the site locally (see [README.md](README.md) for prerequisites and setup steps)

3. **Make your changes** in the `content/` directory

4. **Preview your changes** by running the command `hugo server` and opening `http://localhost:1313`

5. **Commit** your changes with a short, descriptive message

6. **Open a Pull Request** against the `main` branch

## Content structure

All documentation lives under `content/docs/`. Here is the high-level layout:

```
content/
  docs/
    analyzers/
      ApplicationCop/     ← One .md file per rule (e.g. AC0001.md)
      DocumentationCop/
      FormattingCop/
      LinterCop/
      PlatformCop/
      TestAutomationCop/
    getting-started/      ← Installation and configuration guides
```

Each analyzer rule page follows the same structure: a brief explanation of why the rule exists, a "bad" code example, and a "fixed" code example. If you are editing or adding a rule page, matching that pattern keeps things consistent.

## Hugo and Docsy

The site uses [Hugo](https://gohugo.io/) with the [Docsy](https://www.docsy.dev/) theme. If you want to use shortcodes or understand the page layout system, the [Docsy documentation](https://www.docsy.dev/docs/) is the best reference. You do not need to understand Hugo or Docsy in depth to edit content pages.

## Questions

If you are unsure about something before opening a PR, start a [Discussion on GitHub](https://github.com/ALCops/Analyzers/discussions). It is always better to ask first than to spend time on something that does not fit.
