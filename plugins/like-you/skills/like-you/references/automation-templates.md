# Automation templates — the 6 shipped triggers (slide 23)

`automations.md` is **declarative in v0.1**. It is a spec read by future Claude Code sessions, not a live cron daemon. The daemon ships in v0.2.

The grammar is intentionally simple and human-readable. One trigger per block. Indented arrows for the skill(s) to invoke. Optional `(param: value)` after a skill name.

---

## Trigger grammar

```
@every {duration}                  # 1h, 30m, 1d
  → {skill path}

@daily {HH:MM}                     # 24h clock
  → {skill path}

@calendar:meeting-ended
  → {skill path}

@email:new
  → {skill path}

@email:client                      # any sender already in context/people/ with relationship=client
  → {skill path}

@email:from({address-or-domain})
  → {skill path}

@silence {days}d                   # no reply for N days in a thread the user started
  → {skill path}
```

Optional parameters on a skill line: `(voice: yours)`, `(ask-first: true)`, `(from: known-clients)`, `(from: prospects)`. These are passed as flags when the skill is invoked.

---

## The 6 default automations (slide 23)

When Phase 6 writes `automations.md`, lift these blocks. Comment out any whose underlying skill wasn't generated in Phase 5.

### 1. Inbox triage (`@every 1h`)

```
# סריקת אינבוקס — every hour, triage what landed.
# Tags as: reply-now / draft-for-you / wait / archive.

@every 1h
  → skills/judgment/triage-inbox
  → skills/content/reply-email      (from: known-clients, auto-send: false)
  → skills/content/draft-proposal   (from: prospects, ask-first: true)
```

### 2. Meeting followup (`@calendar:meeting-ended`)

```
# סיכום אחרי פגישה — when a calendar event ends, draft the followup.

@calendar:meeting-ended
  → skills/content/meeting-followup
  # → ping-whatsapp ("how did it go?")   # v0.2 — needs WA/Telegram bridge
```

### 3. Auto-reply to known clients (`@email:client`)

```
# תשובה אוטומטית ללקוחות קיימים — for recurring questions, status,
# acknowledgments. Drafts only — never auto-sends in v0.1.

@email:client
  → skills/content/reply-email      (voice: yours, ask-first: true)
```

### 4. New prospect outreach (`@email:new`)

```
# טיוטה לפנייה חדשה — LinkedIn or website inbound. Drafts only.

@email:new
  → skills/content/draft-proposal   (ask-first: true)
```

### 5. No-reply nudge (`@silence 3d`)

```
# תזכורת ללא-מענה — if user-initiated thread is silent 3 days,
# ASK user whether to draft a nudge. Never sends automatically.

@silence 3d
  → skills/content/chase-nudge      (ask-first: true)
```

### 6. End-of-day summary (`@daily 18:00`)

```
# סיכום יום — at 18:00, write today's summary.
# In v0.1 this writes to ~/Business/.claude/_summaries/YYYY-MM-DD.md.
# In v0.2 it can also send to WhatsApp/Telegram.

@daily 18:00
  → skills/content/end-of-day-summary
  # → whatsapp                          # v0.2
```

---

## Filter rules

Phase 6 filters these 6 templates before writing the file:

- If `meeting-followup` wasn't generated in Phase 5 → comment block 2 out.
- If `draft-proposal` wasn't generated → comment blocks 1 (the third arrow) and 4 out.
- If `chase-nudge` wasn't generated → comment block 5 out.
- For block 3, replace `@email:client` only if the user has ≥1 person in `context/people/` with `relationship: client`. Otherwise comment.

---

## How v0.1 "runs" automations

Until the daemon ships, the user has two options:

**A. Manual sweep.** In any Claude Code session, say `"run my automations"`. Claude walks `automations.md` top-to-bottom, fires every block whose trigger condition is currently true (e.g., for `@every 1h`, true if the last run was >1h ago; for `@daily 18:00`, true if today's run hasn't happened and it's past 18:00).

**B. Cron.** The user can wrap option A in their own cron / launchd:
```
0 * * * *  cd ~/Business && claude -p --bare "run my automations"
```
Document this in `automations.md` itself, in a `# How to run` comment block at the top.

---

## Skill invocation parameter passing

When the (future) daemon fires a block, it builds a `claude -p` invocation:

```
claude -p --bare \
  --add-dir ~/Business \
  --append-system-prompt-file ~/Business/.claude/context/voice.md \
  --plugin-dir ~/.claude/plugins/marketplaces/82labs/plugins/like-you \
  "run skill {skill-slug} with input {trigger-context}"
```

Parameters from the trigger (e.g., `(from: known-clients)`) become natural-language qualifiers in the prompt.

This is for v0.2. **Do not implement the runner in v0.1.** The file is documentation that the agent reads when manually invoked.
