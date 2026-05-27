# voice.md template + extraction prompt

This is the most load-bearing file in the workspace. Every content skill reads it before drafting. Failures here cascade into every output.

---

## Template

```markdown
---
generated: {ISO timestamp}
sample:
  gmail: {n_clean} clean / {n_contaminated} excluded
  whatsapp: {optional, if WhatsApp export was provided}
contamination_flagged:
  count: {n_contaminated}
  reasons: [em-dash-density, hope-finds-you-well, bullet-overuse, ...]
language_primary: he | en
language_mix: {0.0-1.0 ratio of Hebrew chars to total}
confidence: high | medium | low
---

# Voice

## TL;DR

{10 lines max. The agent reads ONLY this in the worst case. Make it count.
Cover: typical message length, opening style, sign-off style, register, top forbidden phrases.
Example:
- Sentences average 8 tokens. Maximum 3-sentence paragraphs.
- Open with name + em-dash. No "שלום". No "I hope...".
- Sign-off: first name only, or no sign-off at all (just stop).
- Hebrew primary, English for technical terms.
- Never "act as", "you are a helpful assistant", "thrilled", "delighted", "מקווה שמוצא אותך טוב".
- Always offer a no-pressure escape clause in cold replies.
- Name prices in first reply. Never ask "what's your budget".}

## Verbatim exemplars

5 real emails. **Quote raw — do not paraphrase.** Each gets a header noting audience + context.

### Exemplar 1 — To a known client, mid-project, friendly

```
{paste verbatim email here}
```

### Exemplar 2 — Cold reply to a prospect

```
{paste verbatim email here}
```

### Exemplar 3 — Quick async to a partner

```
{paste verbatim email here}
```

### Exemplar 4 — Polite-firm boundary or refusal

```
{paste verbatim email here}
```

### Exemplar 5 — Longest you tend to write (still tight)

```
{paste verbatim email here}
```

## Sentence rhythm

- Median sentence length: {N} tokens
- p25 / p75: {N} / {N}
- Fragments allowed: yes/no, frequency
- Paragraph length: typically 1-2 sentences
- Multiple periods per "sentence" (stylistic): yes/no

## Openings & sign-offs

Top openings (ranked by frequency in sample):
- `{First name} —` — 62%
- Bare first name + comma + question — 18%
- Plain "hello" or "היי" — 10%
- Other — 10%

Top sign-offs:
- First name only — 71%
- No sign-off, just stops — 22%
- "תודה" — 5%
- Other — 2%

## Punctuation fingerprint

- Em-dashes per 100 words: {N}
- Ellipses: rare / common
- Exclamation marks: {N per 100 words}
- Question density: {%}
- ALL CAPS for emphasis: yes/no

## Code-switching (if multilingual)

When you switch from primary language to secondary:
- Technical terms only (e.g., "MVP", "scope", "deliverable")
- Quoted material
- Whole sentences when addressing English-speakers in a mixed thread

## Jargon you use

Words extracted from your mail that aren't in baseline corpus:
- {top 10-15 user-specific terms}

## Jargon you avoid

Words a generic LLM uses that you never do (flagged by absence):
- "thrilled", "delighted", "amazing"
- "moreover", "furthermore", "in conclusion"
- "I hope this email finds you well"
- "circle back", "touch base"

## Register by audience

### client/

{2 lines summary + 1 exemplar.
Example: "Direct. Mid-formality. Always name the next action. Don't ask permission to send a deliverable — send it."}

### partner/

{2 lines + 1 exemplar.
Example: "Shorthand. Inside jokes okay. Skip the framing — they have it."}

### prospect/

{2 lines + 1 exemplar.
Example: "Curious tone. Ask one question. Offer one concrete value. Always an escape clause: 'no pressure if not now.'"}

### vendor/

{2 lines + 1 exemplar.
Example: "Polite-firm. Specifics: PO numbers, dates, exact line items. No small talk."}

## Intent rules (אותה כוונה, שני קולות שונים)

Behaviors extracted with frequency evidence:

- **What you offer**: "Names a price in 8/10 first replies to prospects. Never asks 'what's your budget'."
- **What you refuse**: "Doesn't send second follow-up if first was ignored for >7 days. Pattern: 0 follow-ups beyond the first in sample."
- **Escape clauses**: "9/10 cold replies end with 'no pressure if not now' or equivalent."
- **CC pattern**: "CCs {partner name} on every deal >₪50k. Doesn't CC on prospect intros."
- **Commitments**: "Commits to dates ('Tuesday EOD'), not outcomes. Never writes 'I guarantee'."

## Forbidden phrases (yours, calibrated)

Phrases the generic LLM defaults to that you never use. Hard-ban for content skills:

- "I hope this email finds you well"
- "מקווה שהמייל מוצא אותך טוב"
- "Looking forward to hearing from you"
- "מצפה לשמוע ממך בקרוב"
- "Please don't hesitate to reach out"
- "אל תהסס/י לפנות"
- "act as", "you are a helpful assistant" (in any system-prompt context)
- "thrilled to", "delighted to", "excited to announce"
- {user-specific additions extracted from contrast with sample}

## Hebrew calque blocklist

Inherits `~/.claude/skills/hebrew-copy/SKILL.md` if installed.
User-specific extras below:

- {patterns extracted from contrast — phrases your sample never uses but generic Hebrew LLM output does}

## Provenance

- Sources: {N} Gmail messages, {M} WhatsApp exports (if provided), {K} excluded for contamination.
- Date range: {oldest ISO} to {newest ISO}.
- Contamination reasons (counts): em-dash density: {n}; "hope finds you well": {n}; bullet overuse: {n}; "moreover/furthermore": {n}.
- If a verbatim exemplar above looks wrong, edit it. The agent imitates these directly.
```

---

## Extraction prompt (used by Phase 4 of SKILL.md)

When the skill runs Phase 4, it sends this prompt to a single LLM call (no subagent — uses current session) along with the 50 clean messages and `_meta/voice.stats.json`:

```
You are extracting a writing voice profile for a specific person from their own sent emails.

Here are their statistics:
{voice.stats.json contents}

Here are 50 of their sent messages, contamination-filtered:
{message dump}

Extract:

1. 5 verbatim exemplars covering the user's range: shortest, longest, most casual, most formal,
   one to a stranger. DO NOT PARAPHRASE. Quote each email raw, exactly as sent. Add a one-line
   header naming the audience and tone.

2. Per-audience register notes (client / partner / prospect / vendor): 2 lines each + 1 exemplar each.

3. Intent rules — behaviors with frequency evidence. e.g. "8/10 first-replies name a price."
   Each rule must cite a count from the actual sample. No vibes.

4. Top 10 jargon words the user uses (not in generic corpus).

5. User-specific forbidden phrases — patterns NOT in their sample but typical of generic LLM output
   in their domain/language. Be specific (full phrases, not categories).

If <30 contamination-free messages remain, label confidence: low and warn explicitly.

Never describe the voice qualitatively ("the user writes concisely") — always cite stats or exemplars.
```
