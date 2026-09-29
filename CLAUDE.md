# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only. There is no `package.json` here and nothing
in this repository executes. It teaches an agent how to build a markdown-backed help center (category folders,
frontmatter articles, static pages, client-side search, locale fallback, tags, validation) in a **Next.js App
Router** app, with the content source as a seam behind one `getHelpIndex(locale)`.

Keep the two straight: the commands and code in `references/` describe the app the agent will generate, not
this repository. The loader, routes, validator script and tests all run in that generated app.

The skill was written by the engineer who has shipped this module; `references/provenance.md` is the
engineering ledger, thirteen entries recording what the audit of the earlier implementation changed and how
the templates verify it, what was kept on purpose, and what was designed in the skill. That file is the
rationale layer: read it before "simplifying" anything.

## Structure

- `SKILL.md`: the entry point, loaded whole on every activation, so it stays at 130 to 160 lines. The
  frontmatter `description` is the trigger surface. The body carries the architecture, the critical facts,
  the hard rules, the quick start, the reference directory table mapping trigger keywords to `references/`,
  and a closing line linking the skills index.
- `references/*.md`: one topic per file, loaded on demand. `adaptation.md` (the seam contract and the
  non-negotiables restated) and `content-model.md` (config and types) are the design entry points; the rest
  cover the loader, search, tags (`tags.md`: slug identity, tag page, chips, tag validation), i18n, routes,
  UI (`ui.md` shell and nav, `ui-content.md` cards, lists, renderer and style hooks), extensions and the
  ledger.
- `README.md`: the human front door, in the fixed section order of the index's STANDARD.md. `CHANGELOG.md`:
  Keep a Changelog, newest first.
- `evals/`: `prompts.md` holds what an operator types after installing, in their words; the first prompt is
  the agent eval run before every release. Every other file there is one eval run: measured frontmatter that
  is never edited, then the notes of the person who ran it. Add a prompt rather than rewording one that has
  results. The procedure is section 10 of the index's STANDARD.md.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, copied verbatim from
  STANDARD.md. Never edit it and never add a trigger.

## Editing conventions

- **Code blocks are compiled.** Every ` ```ts ` / ` ```tsx ` block whose first line is `// file: <path>` is
  copied into a scratch project and type-checked under `strict` and `noUncheckedIndexedAccess`; the
  `*.test.ts` blocks are run. Keep that first line, keep imports complete, and re-run the check after editing
  any block.
- **Identifiers are shared across files.** `HELP_CONTENT`, `HelpIndex`, `getHelpIndex`, `toSummary`,
  `toSearchDoc`, `helpHref`, `helpPath`, `HelpStrings`, `HelpTag`, `tagSlug`, `tagRefs`, `groupTags` appear in
  several references. Rename in all of them or none.
- **Keep three lists in sync with `references/`**: the reference directory table in `SKILL.md`, the quick
  start in `SKILL.md`, and the file table in `README.md`. Links are relative: `[x.md](references/x.md)` from
  `SKILL.md`, `[x.md](x.md)` between references.
- **The odd-looking parts stay.** `hidden` instead of a max-height transition, `toSummary` before client
  props, `?? Number.MAX_SAFE_INTEGER` for order, the untrimmed-query note in search, the `tagSegment` throw in
  `config.ts`, span (not link) chips inside list rows: each is a documented ledger entry or an HTML rule.
  Check `provenance.md` first; a change to a template adds an entry there.
- **Measured numbers are load-bearing.** The ~200-article threshold for full-text search, about a kilobyte
  per search document and the 20-vote floor for feedback aggregates are design parameters; do not restate
  them loosely and do not invent new ones. Corpus counts of the earlier implementation stay out of every file.
- **Additions are marked as additions.** Anything designed in the skill and never run in production goes
  under *Added* in `provenance.md`; new capability goes there too, or in `extensions.md` as a design.
- **The eight non-negotiables are never optional.** They are the hard rules in `SKILL.md`, the list in
  `README.md` and the list in `adaptation.md`, in the same order: summaries to the client, `hidden` not a
  height cap, untranslated pages canonicalised to the default locale, validation in CI, sort by order then
  title then slug, search that trims, tokenises and folds diacritics over tags and headings, chrome strings
  through `HelpStrings`, tags keyed by `tagSlug`. Never present any of them as optional in a reference.
- **The host renames the categories; the frontmatter stays.** Category ids `app`, `general`, `development`
  are deliberately generic and the rename procedure is in `adaptation.md`. Frontmatter field names are the
  authoring contract and are never renamed.
- **The description is the trigger surface.** If the skill's scope changes, update its trigger keywords and
  the *When to use* and *When NOT to use* sections together.
- **Plain punctuation, prose wrapped at 110 columns.** No em-dashes, en-dashes, arrows, middle dots or smart
  quotes anywhere in the markdown, code comments included; a rendered UI glyph is written as a JavaScript
  escape.
