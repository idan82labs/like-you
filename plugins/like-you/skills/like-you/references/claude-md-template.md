# CLAUDE.md template

Fill placeholders in `{curly braces}`. Lines starting with `<!--` are comments — strip before writing.

```markdown
---
generated_by: like-you v0.1.0
generated_at: {ISO timestamp}
use_case: {business | personal | hybrid}
mode: {full | preliminary | cold-start}
business_identity: {primary email if known}
confidence:
  gmail: {n_sent} sent in 90d
  calendar: {n_events} events in 90d
  drive: {n_docs} docs
  sheets: {n_sheets} sheets
  voice_sample: {n_clean} clean / {n_contaminated} contaminated
language: {he | en | mixed}
---

# Workspace constitution

This file is read first by every Claude Code session that runs in this workspace.
If you are a Claude Code agent reading this, **follow these rules before doing anything else.**

## 1. Identity

{One-line answer from the user to "what do you do?"}

## 2. Memory layout

- `context/people/` — one file per person you work with. Read the relevant one before addressing that person.
- `context/companies/` — companies. Linked from people pages.
- `context/projects/` — active projects, status, who's involved.
- `context/voice.md` — your writing voice. **Read this before drafting anything addressed to a human.**
- `automations.md` — declarative list of proactive triggers (v0.1 = spec only, no daemon).
- `raw/` — immutable 90-day snapshot from Gmail/Calendar/Drive/Sheets. **Never write here.**
- `_meta/log.md` — extraction audit trail.

## 3. Voice rule (load-bearing)

Before writing ANY message addressed to a person:

1. Read `context/voice.md`.
2. Read `context/people/{slug}.md` for the addressee, especially the `relationship:` field.
3. Use the audience-specific register block in `voice.md` (`client/partner/prospect/vendor`).
4. Apply the Hebrew calque blocklist if the message is in Hebrew.

Never use these phrases unless they appear verbatim in a `voice.md` exemplar:

- "I hope this email finds you well"
- "מקווה שהמייל מוצא אותך טוב"
- "אל תהסס/י לפנות"
- "Looking forward to hearing from you"
- "Please don't hesitate to reach out"

## 4. Privacy rule

- Never include in any output a label or folder name that appears in `references/skip-tokens.md`.
- Never include data sourced from `raw/` messages whose `labels:` frontmatter contains a skip-list token.
- Treat anything in `context/people/{slug}.md` `notes:` field as user-private — do not re-output it verbatim unless the user explicitly asks for it.

## 5. Commands you can run for me

The user may ask any of these in natural language:

- **"triage"** → run `skills/judgment/triage-inbox`
- **"reply to {name}"** → read `context/people/{slug}.md`, then run `skills/content/reply-email`
- **"prep brief for {calendar event}"** → run `skills/content/meeting-followup` in reverse-prep mode
- **"draft proposal for {company}"** → run `skills/content/draft-proposal`
- **"run my automations"** → walk `automations.md` top-to-bottom, invoke listed skills

## 6. Operating mode caveat

{If mode == full:} All systems normal.
{If mode == preliminary:} Sample size was below `full` thresholds. Outputs labeled `confidence: medium`. Do not generate client-specific advice without flagging uncertainty.
{If mode == cold-start:} Workspace was bootstrapped without Gmail/Calendar data. Voice.md is based on a single pasted sample. Treat context/ as a stub. The user should re-run `like-you` once they have 90 days of history.
```
