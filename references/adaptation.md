# Adaptation contract

Everywhere this module touches its host. Fill in the right-hand column from the
target app **before** generating code, and confirm the category vocabulary with
the user: it is the one thing you cannot infer.

| Seam | This skill ships | Your app supplies | Where it plugs in |
|---|---|---|---|
| Category vocabulary | `app`, `general`, `development` | Its own ids and labels | `config.ts` + `strings.ts` |
| Content location | `content/help`, single or per-locale tree | A path and a layout choice | `HELP_CONTENT` |
| Locale routing | `[lang]` segment, default locale unprefixed | Its proxy/middleware and locale list | `HELP_CONTENT.routeLocales`, `localizePath` in `paths.ts` |
| Strings | Typed table, English | Its i18n dictionaries | `getHelpStrings` in `strings.ts` |
| Tag URLs | `tag` segment, slug folding | A segment that is not a category id; the route folder named to match | `HELP_CONTENT.tagSegment`, [tags.md](tags.md) |
| Markdown rendering | Minimal `react-markdown` component | Its existing renderer | `HelpMarkdown.tsx` |
| Frontmatter parsing | `gray-matter` wrapper | `gray-matter` (or its parser) | `frontmatter.ts` |
| Site identity | `siteName`, `siteUrl` | Its values for JSON-LD, as literals | `HELP_CONTENT` |
| Header slots | `brand`, `headerActions`, `footer` props | Logo, CTA, footer components | `HelpShell` |
| UI primitives | Structure, ARIA, `data-help-*` hooks | Input, drawer, icons, card styles | [ui-content.md](ui-content.md) hook table |
| Styling | Layout utilities only | Design tokens, typography | CSS on the hooks |
| Analytics | `reportHelpSearchMiss(query)` | Its event pipeline | `analytics.ts` |
| Tests | `vitest`-style files | `vitest` or `bun test` | `*.test.ts` |
| Validation | `validate:help` script | `prebuild`, or a CI step before `build` | `package.json` |

No tenant scope, no auth guard, no database, no object storage: the module is
public, read-only and file-backed. If the host wants a private help center,
that is the host's route protection in front of these pages, and nothing
inside the module changes.

## Host probe

Run before writing anything:

```bash
grep -E '"(next|react|gray-matter|react-markdown|remark-gfm|next-intl|vitest)"' package.json
ls src/app/*/ 2>/dev/null | head          # locale segment? route groups?
ls src/lib/*markdown* src/components/*/Markdown* 2>/dev/null   # existing renderer
grep -rn "localizePath\|getLocalizedPath\|localePrefix" src/lib | head -3     # link helper
ls content/ 2>/dev/null                   # where other markdown lives
cat CLAUDE.md AGENTS.md 2>/dev/null | head -60
```

Then read the host's closest markdown-backed sibling end to end. Copy its
loader's file layout, its naming, where it keeps types, and how its pages call
`generateMetadata`.
Consistency inside one codebase beats the skill's conventions.

**Add no dependency the host lacks without asking.** `gray-matter`,
`react-markdown` and `remark-gfm` are the only runtime packages the templates
use; a host with its own frontmatter parser and renderer needs none of them.
Working unattended, with nobody to ask, install them and `vitest` from npm and
list them in the handover. The package registry is not an external service: a
note that none is reachable never means writing a parser, a renderer or a test
shim in their place.

**Site identity is two literals.** Take `siteName` and `siteUrl` from the
host's existing site config (`metadataBase`, a site constant) into
`HELP_CONTENT`. A host with none keeps the placeholders and says so in the
handover. The module reads no environment variable; do not add one for either.

## Naming the categories

Category ids are folder names, URL segments and string keys at once. Decide
them once, before generating:

| Canonical | Product docs | Developer platform | Internal wiki |
|---|---|---|---|
| `app` | `guides`, `using-the-app` | `getting-started` | `how-to` |
| `general` | `concepts`, `learn` | `concepts` | `policies` |
| `development` | `developers`, `api` | `api`, `sdks` | `engineering` |

Rename in `HELP_CATEGORY_IDS`, the `categories` block of every strings table,
and the folder names. Nothing else references them by literal. Changing an id
later is a URL change, so add redirects.

**Do not rename** `slug`, `order`, `tags`, `related`, `updatedAt`, `featured`,
`translated`: they are the frontmatter contract authors write against.

## What is not negotiable

The same eight, in the same order, as the hard rules in `SKILL.md` and the non-negotiables in `README.md`.

1. **Summaries, never articles, cross to the client.** `toSummary` /
   `toSearchDoc` exist so the corpus is not serialised into every page.
2. **`hidden`, not a height cap, on collapsible nav.** A `max-height` cap with
   `overflow-hidden` clips whatever does not fit and reports nothing.
3. **Untranslated pages canonicalise to the default locale** and stay out of
   hreflang and the sitemap. Otherwise every fallback page is a duplicate.
4. **Validation runs in CI.** The runtime is deliberately forgiving (drops bad
   refs, sorts missing orders last); the script is where authors find out,
   including that a `related` slug was renamed.
5. **Sort by `order`, then title, then slug.** Ties left to `readdir` order
   differ between filesystems.
6. **Search trims, tokenises and folds diacritics, over tags and headings as
   well as title and description, ranked by field.** A query is matched
   however it is typed.
7. **Every chrome string goes through `HelpStrings`.** The header, footer and
   notices then switch locale together with the host.
8. **Tags are keyed by `tagSlug`, never by spelling.** Case, hyphens and
   plurals drift across authors; every variant lands on one page and the
   validator reports the drift.

Everything else, meaning styling, naming, renderer and i18n system, is the host's.

## Integration points

Nothing fails loudly without these:

- [ ] `/help` link in the host's header/footer navigation
- [ ] Locale proxy/middleware rewrites `/help/*` to `/{default}/help/*` (or the pages live outside the locale segment)
- [ ] `help` strings added to **every** locale dictionary, not just the default
- [ ] `helpSitemapEntries()` merged into the sitemap
- [ ] Redirects from a legacy help subdomain or old article URLs
- [ ] `validate:help` as `prebuild` (or a CI step before `build`), and run once on the existing corpus first; expect tag-spelling warnings
- [ ] `tag/` route folder named to match `HELP_CONTENT.tagSegment`; no category uses that id
- [ ] `reportHelpSearchMiss` wired to analytics
- [ ] Support/contact CTA placed on the landing page (a host component, passed as children)
- [ ] `<html lang>` still set by the host; article body gets `lang={article.locale}`

## Order of work

Config, then model, then frontmatter and loader, then the validation script
(run it on the real content now: it finds authoring problems before any UI
exists), then strings, paths and SEO, routes, shell and navigation, search,
tests, and last the host styling.

Run the loader and validator against the actual corpus as soon as they exist.
Real content is the only fixture that finds the `order` ties, the stale dates,
the renamed slug that three `related` lists still point at.
