# Privacy hard-skip tokens

The skill skips any source whose `labels:`, folder path, or filename matches a token in this list (case-insensitive, substring match). Tokens are matched against:

- Gmail label names
- Drive folder paths
- File names
- Email subject lines (for marginal cases — adds a `sensitive: true` flag but doesn't skip outright)

The skip is enforced **before** any LLM call sees the content. Phase 1 (ingest) checks this; Phase 2 (extract) re-checks; CLAUDE.md instructs every downstream session to re-check at output time.

This is a hard rule, not a soft preference.

---

## English tokens

```
legal
medical
finance
banking
tax
accounting
hr
human-resources
payroll
salary
benefits
insurance
personal
private
family
divorce
custody
therapy
psychologist
psychiatrist
health
diagnosis
prescription
confidential
nda
embargoed
attorney-client
privileged
```

## Hebrew tokens (עברית)

```
חוקי
משפטי
משפט
עורכ-דין
עו"ד
רפואי
רפואה
בריאות
טיפול
פסיכולוג
פסיכולוגיה
פסיכיאטר
תרופה
מרשם
אבחון
כספים
כספי
כסף
בנק
משכורת
שכר
מס
מסים
ביטוח
משאבי-אנוש
אישי
פרטי
משפחה
משפחתי
ילדים
גירושין
משמורת
סודי
חסוי
```

---

## Domain hard-skips (regardless of label)

Any participant in a thread whose email address ends in one of these → skip the whole thread.

```
# Israeli health funds (kupot cholim)
clalit.org.il
maccabi4u.co.il
meuhedet.co.il
leumit.co.il

# Generic medical/legal-sounding TLDs and patterns
*.law
*.med
*.gov.il      # government correspondence — extra caution; skip by default

# Banks (Israeli)
mizrahi-tefahot.co.il   # NOTE: user-configurable — if user works WITH the bank, they'll uncomment
hapoalim.co.il
leumi.co.il
discountbank.co.il
fibi.co.il
```

The bank list is interesting: **for the deck's intended user**, mizrahi-tefahot may be a *client*, not a private banking relationship. So `mizrahi-tefahot.co.il` should NOT be on the default skip list — but other banks might be. This file ships with the bank skips commented out. The skill asks at pre-flight:

> "Do you work WITH any banks as clients? If yes, name the domains so I include them."

Capture answer in `_meta/skip-overrides.json`.

---

## Marginal cases — flag but don't skip

If a label or folder contains one of these "soft" tokens, add `sensitive: true` to the source frontmatter but ingest the content. The user-facing rule: the skill won't include sensitive-tagged content in any drafted output without the user explicitly confirming.

```
hr-internal     # internal company HR, not external counsel
finance-team    # ditto
client-private  # client mentioned this is private to them
ndaed
under-review
```

---

## How the user extends this list

The skip list is a reference file inside the marketplace plugin (immutable from the user's perspective). To extend per-vault, the user creates `~/Business/.claude/skip-overrides.md` with additional tokens, one per line. The skill merges this list at ingest time.

Format:
```
# ~/Business/.claude/skip-overrides.md
# One token per line. Comments start with #.
# Added 2026-05-27 — internal use only.

acquisition-talks
m-and-a
board-private
```

The skill creates this file as a stub on first run with no tokens, just the header — making the override path discoverable.

---

## Audit trail

Every skipped source is logged to `_meta/skipped.md` with:
- Source identifier (gmail thread ID, drive file ID, calendar event ID)
- Reason (which token matched)
- Date

The user can review what was excluded and override if needed by adding an `--include-token={token}` flag and re-running ingest. This is intentional friction — easier to skip than include.
