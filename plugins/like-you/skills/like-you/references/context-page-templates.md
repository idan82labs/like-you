# context/ page templates

Three entity types: people, companies, projects. One file per entity. Cross-link with relative paths, not wikilinks.

## context/people/{slug}.md

```markdown
---
name: {Canonical Display Name}
slug: {ascii-kebab-slug}
emails: [primary@example.com, secondary@example.com]
phone: {optional}
company: {company-slug or null}
title: {optional}
relationship: client | partner | prospect | vendor | colleague | unknown
first_seen: {ISO date}
last_seen: {ISO date}
interactions: {integer count}
aliases: [optional list of alternate names found in mail]
language: he | en | mixed
confidence: high | medium | low
---

## Who they are

{1-3 sentences from extracted data. Cite source counts. No LLM speculation.}

Example:
"Co-founder of Mizrachi Bank's innovation arm. Met you on the Sagi MVP project. 12 threads in raw/gmail/, 4 calendar events in last 90 days."

## How you talk to them

{Lifted from voice.md's register section if it differs from default.
Example: "Short replies. Hebrew. No sign-off. Often a single question, then a price."}

## Active work

{Reference project pages by relative path.
Example:
- See [context/projects/sagi-mvp](../projects/sagi-mvp.md)
- See [context/projects/mizrachi-platform](../projects/mizrachi-platform.md)}

## History

{Optional: 3-5 bullet points of significant past interactions, chronological.
Pre-seeded from raw/. User-editable.}

## Notes

{User-private. Do not include in any output unless explicitly asked.}
```

---

## context/companies/{slug}.md

```markdown
---
name: {Company Name}
slug: {ascii-kebab-slug}
domain: {primary-domain.com}
domains: [list of all observed]
industry: {optional, derived from sigs/Drive}
people: [list of person slugs at this company]
first_seen: {ISO date}
last_seen: {ISO date}
status: active | dormant | unknown
---

## What they do

{2 lines if derivable from Drive content or email signatures, else just the contact list.}

## People here

- [name1](../people/name1.md) — {title or role}
- [name2](../people/name2.md) — {title or role}

## Active projects with them

- [project-slug](../projects/project-slug.md)
```

---

## context/projects/{slug}.md

```markdown
---
name: {Project Name}
slug: {ascii-kebab-slug}
status: active | dormant | done | unknown
last_touch: {ISO date}
people: [slug list]
companies: [slug list]
deliverable: {optional one-liner}
confidence: high | medium | low
---

## What it is

{3-line description if extractable from raw/ titles/content, else just refs to people and companies.}

## Status

{What's currently open, what's blocked, what's next. Pre-seeded from latest emails/events; user-editable.}

## Cadence

{If detectable: weekly meeting? Async-only? Slack-driven?
e.g., "Tuesday 14:00 standup, 30min. Async between."}
```

---

## Slug rules

- ASCII only. Lowercase. Hyphens for spaces.
- For Hebrew names: transliterate via `python-slugify` with `lang_code='he'` if available, else use `{8-char-hash-of-canonical-name}` with original name in frontmatter `name:`.
- Resolve collisions by appending `-{2-char-suffix}` (`-co`, `-il`, etc.) not numeric — easier for humans to recognize.

## Cross-link convention

Always relative path from current file: `[label](../people/dan-cohen.md)`. Never absolute. Never wikilinks (`[[name]]`) — keeps the vault portable across tools, no Obsidian dependency.

## Confidence levels

- **high** — ≥5 distinct source mentions across ≥2 sources (e.g., Gmail + Calendar).
- **medium** — ≥3 mentions in 1 source, OR ≥2 across 2 sources.
- **low** — single source, single mention. Likely auto-extracted, may be wrong.

Low-confidence entities get a banner in their body:
`> ⚠️ Auto-extracted from a single source. Review before relying on this page.`
