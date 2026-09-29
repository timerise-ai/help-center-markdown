---
prompts:
  - prompt: "Add a help center to this Next.js app from markdown files in content/help: category pages, article pages, search, and a sitemap entry for every article."
    stack: Repository files
  - prompt: Our docs are markdown files in English and German. Build /help with a locale fallback, so an article missing in German shows the English one instead of a 404.
    stack: Repository files
  - prompt: "Audit our help center: search misses articles, categories sort inconsistently and some tags are spelled two ways."
---

# Prompts

What an operator types after installing this skill, in their own words. An agent eval installs the skill
into an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks, builds
and tests the result; the first prompt runs before every release. The results are the other files in this
folder. Section 10 of [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) says how a
run is made. The prompts and the newest runs are on
[the skill's page](https://timerise.ai/skills/help-center-markdown) on timerise.ai.
