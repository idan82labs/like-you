# Hebrew calques — bureaucratic patterns to block

When `voice.md` is generated and the user's primary language is Hebrew (or mixed), inherit this blocklist. Add user-specific extras detected from contrast with their actual sent mail.

The point of slide 16 of the talk: same intent, two voices. Generic LLM Hebrew sounds translated. Your actual voice is short, direct, often fragmentary. This file is the active rule set that bridges the gap.

---

## Hard bans (calques of English idioms into Hebrew)

These phrases get translated from English into Hebrew by default LLM behavior. They sound bureaucratic and "AI-written" to Israeli ears. Never produce them in content skill output.

| Banned | Why | Use instead |
|---|---|---|
| `מקווה שהמייל מוצא אותך טוב` | Direct calque of "hope this email finds you well." | No greeting. Open with name + dash, or jump in. |
| `אל תהסס/י לפנות אליי` | Bureaucratic. Calque of "don't hesitate to reach out." | `תכתוב לי`, `דבר איתי`, or just stop. |
| `מצפה לשמוע ממך בקרוב` | Calque of "looking forward to hearing from you." | Don't add a closer. Stop after the question. |
| `מקווה שעבר עליכם סופ"ש נעים` | Calque of "hope you had a good weekend." | Skip entirely. |
| `אשמח לפנות אליכם` | Bureaucratic. | `אכתוב לכם`, or just send. |
| `על מנת` | Legal-Hebrew register. | `כדי ש-` or just the verb. |
| `מאחר ו-` | Old-fashioned. | `כי`, `מכיוון ש-`. |
| `אשר` (as a relative pronoun) | Stiff. | `ש-`. |
| `הנני` | Archaic, bureaucratic. | First-person verb directly. |
| `אבקש להסב את תשומת לבכם` | Bureaucratic. | `שימו לב ל-`. |
| `בזאת אני מאשר` | Legal register. | `מאשר`, or `כן`. |
| `נשמח לסייע` | Service-desk template. | Specific offer. |

---

## Calque structures (not single phrases — full patterns)

- **`כדי ל-{infinitive}`** when `כדי ש-` or a bare verb works.
  - ❌ `נפגשנו כדי לוודא שהפרויקט בסדר` (calque of "to verify")
  - ✅ `נפגשנו לוודא שהפרויקט בסדר` (just the verb)
- **`מצב של {noun}`** when a verb works.
  - ❌ `מצב של תקלה` → ✅ `תקלה`
- **`לבצע {verb-noun}`** when a verb works.
  - ❌ `לבצע בדיקה` → ✅ `לבדוק`
- **`אנו מציעים לכם`** instead of `אנחנו מציעים לכם` (or just `הצעה`).

---

## Pattern-interrupt openings (preferred)

Israeli founders open emails with:

- `דן —` (name + em-dash)
- `דן, שאלה קצרה.` (name + comma + topic)
- `דן.` (just the name)
- `היי דן,` (only for partners/friendly; never to a cold prospect)
- For very short replies, no opening at all — jump straight to content.

Do NOT use:
- `שלום דן,` followed by a paragraph of pleasantries.
- `שלום וברכה`.
- `לכבוד {name}` unless explicitly to a government/legal recipient.

---

## Sign-off conventions

Most Israeli founder mail signs off with:

- First name only on a new line.
- Nothing — just stops after the last sentence.
- `תודה` (terse, often when asking for something).

Avoid:
- `בברכה רבה`, `כל טוב`, `בכבוד רב` (formal register, sounds aged).
- `בברכה,` as a default sign-off (overused by templates).
- Long email signatures with title/company/phone/social. Use a flat one-liner or nothing.

---

## Code-switching rules

When the user mixes Hebrew and English (which most Israeli tech founders do), keep it natural:

- Technical terms stay in English: `MVP`, `roadmap`, `deliverable`, `scope`, `KPI`, `runway`, `bug`, `PR`, `commit`, `endpoint`.
- Whole sentences in English when addressing an English-speaker in a mixed thread.
- Quoted material in original language.
- DO NOT translate technical terms into Hebrew — `אבן דרך` for "milestone" reads as forced.

---

## When voice.md says "this user is more formal than baseline"

If the user's actual sent mail uses some of these "banned" phrases (e.g., they really do open with `שלום` to clients), preserve the user's voice — these bans apply to generated content where the LLM would default to them, not to the user's authentic style. The contamination filter in Phase 2 already excludes LLM-assisted mail, so what remains should be authentically the user's.

Rule: a phrase is banned only if it does NOT appear in the user's ≥5 clean exemplars.

---

## User-specific extras (filled in at generation time)

Phase 4 adds detected user-specific bans below. Examples:

```
- "thrilled" — not in your sample, common in LLM output for product launches.
- "מצפה לשמוע" — not in your sample even once, but every generic Hebrew LLM output ends with it.
- "circle back" — corporate jargon, never in your sample.
```

These extras are computed by contrasting the user's top words against a baseline Hebrew/English business-mail corpus.
