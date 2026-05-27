# Skill catalog — what to generate and when

Phase 5 of `SKILL.md` walks this catalog and picks 5–10 skills based on what the user's data actually shows.

## Rules

1. **Hard cap**: 10 generated skills.
2. **Floor**: 5 skills. Even cold-start gets the universal set.
3. **Always-on**: `reply-email`, `triage-inbox`, `prioritize`, `end-of-day-summary` (the 4 universals).
4. **Conditional**: the rest. Each has a `precondition` evaluated against `_meta/entities.json`.
5. **Per-client skills** (`respond-as-me-{client}`): cap at 2 total, only top-quintile interaction frequency clients with ≥10 sent emails.
6. **Print the proposed catalog to the user before generating**. Let them veto.

## Catalog

| Skill | Folder | Always? | use_case | Precondition |
|---|---|---|---|---|
| `reply-email` | content | ✅ yes | business + personal + hybrid | none |
| `triage-inbox` | judgment | ✅ yes (business + hybrid) | business + hybrid | none |
| `prioritize` | judgment | ✅ yes | business + personal + hybrid | none |
| `end-of-day-summary` | content | yes (business + hybrid) | business + hybrid | none |
| `meeting-followup` | content | no | business + hybrid | ≥5 calendar events with external attendees in 90d |
| `draft-proposal` | content | no | business + hybrid | ≥2 files in Drive with titles matching `proposal\|quote\|הצעה\|הצעת מחיר\|sow` |
| `design-deck` | content | no | business + hybrid | ≥1 file with mimeType `application/vnd.google-apps.presentation` OR title matches `deck\|presentation\|מצגת` |
| `when-to-escalate` | judgment | no | business + hybrid | ≥3 distinct sender→reply-latency patterns (≥5 messages each) detected |
| `chase-nudge` | content | no | business + hybrid | ≥2 follow-up emails detected (same recipient, ≥3 days apart, no reply between) AND user's voice.md intent rule allows nudging |
| `respond-as-me-{client}` | content | no | business + hybrid | top-quintile interaction frequency client AND ≥10 sent emails to that client. Cap: 2 total. |
| `meeting-brief` | content | no | business + hybrid | ≥3 calendar events with external attendees in NEXT 14 days. Outputs: 1-page brief for an upcoming meeting (attendees, context, open items, suggested questions). Reads `context/people/` + `context/projects/` of attendees. |
| `schedule-meeting` | content | no | business + hybrid | Calendar MCP connected AND ≥10 events in 90d. Outputs: 2-3 candidate slots + draft invite + awaiting-confirm flag. Voice-aware (terse vs. polite per addressee's relationship). |
| `write-spec` | content | no | business + hybrid | ≥2 Drive files with titles matching `spec\|אפיון\|דרישות\|requirements\|brief\|MVP`. Outputs: multi-page spec doc using design-system.md branding. Reads past specs to lift the user's spec structure. |
| `family-bills` | content | no | personal + hybrid | ≥3 invoices/bills in inbox in 90d (utility/insurance/medical billing) |
| `schedule-personal` | content | no | personal + hybrid | ≥5 personal calendar events with family/friends contacts |
| `quick-reply` | content | no | personal | ≥10 short threads (<3 messages) to friends/family contacts |

## use_case filter (applied BEFORE precondition scoring)

If a row's `use_case` column doesn't include the user's `use_case` from Phase 0.4, **skip that skill entirely** — don't even evaluate precondition. This makes `personal` a tight pipeline that won't accidentally generate business skills just because the precondition data is there (e.g., the user happens to have a proposal in their Drive).

After the filter:
- `business` → up to 10 skills generated (existing behavior)
- `personal` → 2-4 skills only (floor 2 ceiling 4). The always-on set is `reply-email` + `prioritize`. `family-bills` and `schedule-personal` are the conditional adds.
- `hybrid` → up to 10, each tagged with `scope_filter:` for runtime invocation

## Scoring (if more than 10 candidates qualify)

Score = `precondition_strength × evidence_count × user_preference_bias`

- `precondition_strength`: 1.0 if precondition has clear signal, 0.5 if marginal.
- `evidence_count`: log10(matches). E.g., 100 proposal-related items in Drive → log10(100) = 2.
- `user_preference_bias`: 1.5 if the user explicitly mentioned this workflow in Phase 0.4 orientation, else 1.0.

Sort descending, take top-(10 minus always-on count).

## When NOT to generate a skill even if precondition passes

- **Sample contamination >60%**: the user's recent sent mail is mostly LLM-assisted. Generated skills would imitate the LLM, not the user. Skip and tell the user.
- **Single-source signal**: precondition passed from only one file/event. Likely a one-off, not a pattern. Skip.
- **Conflicts with voice.md intent rule**: e.g., `chase-nudge` requires the user actually nudges. If `voice.md` says "never sends second follow-up", skip.

## Calibration hints to embed in each generated skill

When writing each `SKILL.md`, include in the body:
- The precondition that triggered generation (so the user can re-evaluate later).
- The N matches that fired it.
- 2-3 exemplars from `raw/` showing the pattern (excerpted, ≤80 chars each).
- A pointer to `voice.md` and the relevant `context/people/` slugs.

## Re-generation

`like-you --recalibrate` re-runs Phase 5 against the current `raw/` and updates skills in place, preserving any user edits in a `## User additions` section at the bottom of each SKILL.md (the regenerator must not clobber this section).
