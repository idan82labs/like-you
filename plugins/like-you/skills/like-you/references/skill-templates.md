# Skill templates — CONTENT and JUDGMENT

Every generated skill is a folder under `skills/content/` or `skills/judgment/` containing one `SKILL.md`. Auto-discovered by Claude Code when the workspace is opened.

The split matters (slide 20 of the talk): **most agent tooling generates only content skills.** Judgment skills are what make the system *act* like the user.

---

## CONTENT skill template

Content skills produce artifacts: a draft email, a proposal, a deck outline, a meeting brief.

```markdown
---
name: {skill-slug}
description: {When Claude should trigger this skill. Specific verbs + objects. Include Hebrew triggers if user is Hebrew-primary.}
---

# {Skill Name}

## What this produces

{1 sentence describing the artifact. e.g., "An email reply addressed to a specific person, in your voice, ready to send."}

## Read before doing anything

1. `~/Business/.claude/context/voice.md` — your voice profile. Use the register block matching the addressee's `relationship:` field.
2. `~/Business/.claude/context/people/{slug}.md` — the addressee (resolve from input).
3. Optionally `~/Business/.claude/context/projects/{slug}.md` — any referenced project.

## Procedure

1. {Step 1 — what to read or compute first}
2. {Step 2 — what to draft}
3. {Step 3 — apply voice + forbidden phrases check}
4. {Step 4 — output format}

## Output format

{Show the exact shape the user will see. Be specific.
Example:
```
Subject: {one line, lowercase, no buzzwords}

{body — short paragraphs, your voice}

{sign-off per voice.md — first name only or nothing}
```
}

## Constraints

- Never use phrases from `voice.md`'s forbidden list.
- Never sign with "Best regards" or "Looking forward".
- Keep the body under {N} words unless the input warrants more.
- If you can't find the addressee's `context/people/` file, ASK before drafting — do not generate against a generic template.

## Examples

{1-2 input → output examples drawn from real raw/ emails. Anonymize if needed.}
```

---

## JUDGMENT skill template

Judgment skills produce **decisions**, not artifacts. Their output is a verdict (`yes/no/wait/escalate/...`) plus a one-line reason citing actual context.

```markdown
---
name: {skill-slug}
description: {When Claude should trigger this. Decisions only — no drafts.}
---

# {Skill Name}

## What this decides

{1 sentence. e.g., "For each open inbox thread, return one of: reply-now / draft-for-you / wait / archive, with reason."}

## Read before deciding

1. `~/Business/.claude/context/people/` — for relationship lookups.
2. `~/Business/.claude/context/projects/` — for project state.
3. `~/Business/.claude/_meta/log.md` — to know what was previously decided.

## Decision rules

These are calibrated against the user's actual sent-mail patterns (from `_meta/voice.stats.json` and `raw/`).

{Concrete rules with thresholds. e.g.:
- IF sender's `context/people/` `relationship:` is `client` AND last reply latency was <2h → `reply-now`.
- IF sender is unknown AND domain is in user's company list → `draft-for-you`.
- IF >7 days since user's last outbound on this thread AND no incoming → `wait` (do not chase, per voice.md intent rule).
- IF subject matches `support|help|broken|outage` AND relationship is `client` → `escalate` + notify user.}

## Output format

```
Thread: {subject}
From: {sender}
Verdict: {reply-now | draft-for-you | wait | archive | escalate}
Reason: {one line citing the rule that fired and the source page}
```

## What to do AFTER deciding

- For `draft-for-you`: chain to `skills/content/reply-email`. Do not auto-send.
- For `escalate`: print to user; do not act.
- For `wait`: log to `_meta/log.md` with revisit date.
- For `archive`: log only.

## Calibration

This skill was generated from {N} sent-mail patterns. Update its rules by editing this file directly.
Run `like-you --recalibrate` to regenerate from updated sent mail.
```

---

## Skill-specific guidance

### `reply-email` (content, always)

The workhorse. Procedure: identify addressee → load people/voice → draft → forbidden-phrase scan → present.

### `draft-proposal` (content, conditional)

Trained from past proposals in Drive. Procedure: identify company → load past proposals → load voice → draft with your standard price/scope structure. Keep numerical anchors from past proposals as memory cues.

### `meeting-followup` (content, conditional)

Triggered after a calendar event ends OR on user request "follow up on {event}". Procedure: read calendar event description + attendees → identify topics from context/projects/ → draft 1-page recap with: what was decided, what's next, who owes what by when.

### `design-deck` (content, conditional) — visual-aware

**This skill produces actual visually-faithful HTML/PDF output, not a markdown outline.** It reads `context/design-system.md` first and reuses the persistent assets in `raw/visual/assets/` rather than regenerating them.

**Required reads (in order):**
1. `~/Business/.claude/context/design-system.md` — colors, typography, layout patterns, asset paths.
2. `~/Business/.claude/context/voice.md` — for headlines and body copy register.
3. `~/Business/.claude/context/projects/{slug}.md` — the topic the deck is about.

**Procedure:**
1. Resolve the topic. Identify audience (`relationship:` from people page).
2. Load tokens from `design-system.md` into a local CSS variable block.
3. Generate a 5–10 slide outline (titles + body bullets), in the user's voice.
4. Render each slide as an HTML `<section class="page">` block:
   - Banner = `<img src="{relative-path-from-output-to-raw/visual/assets/banner.png}">`. **Do NOT inline base64 — use the file.** Copy the asset next to the output HTML if needed.
   - Headers use the `--header` color from design-system.md.
   - RTL/LTR per `direction:` field.
   - Page sizing: `width: 210mm; height: 297mm; @page { size: A4; margin: 0; }`.
5. After generating, run a self-check: every slide has the banner, every header uses `--header`, no forbidden visual moves from § Forbidden visual moves in design-system.md.
6. Save HTML next to a copy of the banner asset. Optionally render to PDF via headless Chrome:
   ```
   "Google Chrome" --headless --disable-gpu --allow-file-access-from-files --print-to-pdf=out.pdf file://out.html
   ```
7. Print to user: path to the HTML + path to the PDF, plus a one-line "compare to {extraction_source} to verify brand fidelity."

**Output format:**
- One HTML file at `{workspace}/outputs/decks/{topic-slug}-{ISO-date}.html`
- One copy of the banner asset adjacent to the HTML (`banner.png`)
- Optionally one rendered PDF at the same path with `.pdf` extension.

**Constraints:**
- The banner is a FILE, never CSS. If the file is missing, halt and ask the user to re-run Phase 2.5.
- Headers MUST be `--header` color (not `--ink`, not arbitrary). This is the load-bearing brand signal.
- No forbidden moves from `design-system.md`'s § Forbidden visual moves.
- If `direction: rtl`, wrap any English fragments in `<span dir="ltr">` to avoid bidi flips.

### `draft-proposal` (content, conditional) — visual-aware

Same visual pipeline as `design-deck` above, but for proposals (longer-form, with sections + a pricing table). Required reads + procedure + output format are identical, but the layout convention is from `design-system.md` § Layout patterns → "Document/proposal layout" (typically: banner top, date upper-left/right, h1 title, body sections, optional pricing table, signature block, page number bottom-center).

Common addition: a pricing table that uses `--header` color for table headers and totals, hairline `--rule` for separators, `--paper-soft` for the surrounding card.

### `chase-nudge` (content, conditional)

For unanswered threads ≥3 days. Procedure: check voice.md intent rule — does this user follow up at all? If no, skip + log. If yes, draft a 2-line nudge in their voice. **Never auto-send.** Per slide 23: "כותב את הניסיון" — drafts, doesn't send.

### `end-of-day-summary` (content, always)

Procedure: scan today's calendar + sent mail + inbox → write 3 lines per slide 23: what happened, what's stuck, what tomorrow needs.

### `triage-inbox` (judgment, always)

The judgment workhorse. See template above for rule examples.

### `when-to-escalate` (judgment, conditional)

For each open thread, decide: notify user vs. let sit. Rule: compare current latency to user's historical latency for this sender. If the user typically replies within {X}h and {X+threshold}h have passed → escalate.

### `prioritize` (judgment, always)

For today's open items (inbox + calendar + manual list), return ranked top-5 with one-line "why now" each. Reads project status and relationship recency.
