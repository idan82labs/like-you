# like-you

> סוכן שכותב, עונה, ומעצב **כמוך** — ומריץ את העסק בזמן שאתה ישן.
>
> An agent that writes, replies, and designs **like you** — and runs the business while you sleep.

One-shot bootstrap of a `~/Business/.claude/` workspace that turns 90 days of your Gmail, Calendar, Drive, Docs, and Sheets into:

- `CLAUDE.md` — the constitution every future Claude Code session reads first
- `context/` — one file per person, company, and project you actually work with
- `context/voice.md` — your writing voice, distilled from 50 real sent emails
- `skills/` — 5–10 business-specific skills generated from your actual work, split into **content** (draft, reply, design) and **judgment** (triage, escalate, prioritize)
- `automations.md` — a declarative spec for proactive triggers (`@every 1h`, `@calendar:meeting-ended`, `@email:from(...)`, `@daily 18:00`)
- `raw/` — immutable snapshot of the 90-day source dump

---

## Install (Claude Code)

```
/plugin marketplace add idan82labs/like-you
/plugin install like-you@82labs
```

Then in any Claude Code session, just say:

```
bootstrap my business
```

or in Hebrew:

```
תקים לי את הסוכן
```

The skill will run pre-flight, ask three orientation questions, and produce the workspace in ~15 minutes.

---

## Prerequisites

- **Required**: Gmail MCP + Calendar MCP connected to Claude Code, OR local-file equivalents (`.mbox` from Google Takeout, `.ics` calendar export).
- **Recommended**: Drive, Docs, Sheets MCPs (Drive folder paths also accepted).
- **Minimum sample**: ≥30 sent emails and ≥10 calendar events in the last 90 days for `full` mode. Below that the skill drops to `preliminary` or `cold-start` mode and labels output accordingly.

If you have no MCPs at all, the skill accepts a folder of local files instead. See `SKILL.md` for the exact fallback flow.

---

## What runs where

| | |
|---|---|
| **Workspace** | `~/Business/.claude/` (path is configurable; defaults to `~/Business`) |
| **Data** | Your machine. Nothing uploaded. `raw/` is immutable; `context/` and `skills/` are rewritable by you. |
| **Voice** | Distilled from your own sent mail. Stays in `context/voice.md`. |
| **Skills** | Generated SKILL.md files in `skills/content/*/` and `skills/judgment/*/`. Auto-discovered by Claude Code. |

---

## Design philosophy

Three layers, one job each:

1. **CLAUDE.md** — the constitution. The rule every session reads first.
2. **`/context`** — the memory. Slow to change, built once, edited rarely. One page per entity.
3. **`/skills`** — the capabilities. Fast to change. Each skill reads the context, applies your voice, and produces output.

The differentiator vs. generic AI tooling is the **judgment** skills — `triage-inbox`, `when-to-escalate`, `prioritize` — which return a decision (`yes/no/wait/escalate`) with a reason citing your actual relationships and project state. Most tools build only content skills.

---

## Honest caveats

- **Voice contamination.** If you've been writing with an LLM for the last 90 days, the extracted voice reflects the LLM as much as you. The skill detects this (em-dash density, "I hope this email finds you well", structured bullets) and excludes contaminated samples with an explicit log.
- **Hebrew calque detection** is opinionated. The `voice.md` ships with a blocklist of bureaucratic Hebrew patterns (`מקווה שהמייל מוצא אותך טוב`, `אל תהסס/י לפנות`, `כדי ל-` calques). Edit if your audience expects formal register.
- **Privacy filters** hard-skip labels and folders containing legal/medical/finance/HR tokens, in English and Hebrew. The skip-list is in `references/skip-tokens.md` and is user-extendable.
- **`automations.md` is a declarative spec in v0.1.** The trigger daemon that actually runs the rules ships in v0.2. For now, the file is read by Claude Code sessions to know what *should* run, but execution is manual or via a separate scheduler.
- **First-run entity merge is imperfect.** Plan to spend 10–15 min after bootstrap reviewing `_meta/log.md` and fixing wrong merges. Aliases go in entity frontmatter.

---

## License

MIT. See [LICENSE](./LICENSE).

## Author

[82Labs](https://82labs.com) — Idan Tal
