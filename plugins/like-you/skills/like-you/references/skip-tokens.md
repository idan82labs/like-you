# Privacy hard-skip tokens

The skill skips any source whose labels, folder path, filename, **subject line**, **sender/recipient domain**, or **body first 500 chars** matches a rule in this file (case-insensitive substring match for tokens; regex match for domain patterns).

**IMPORTANT — many Gmail MCP implementations only expose IMAP-style labels (`[Imap]/Sent`, etc.) and hide user-defined labels.** This means label-matching alone is not sufficient defense. The skill MUST also match against:

- Gmail label names (where exposed)
- Drive folder paths
- File names
- **Email subject lines** (HARD skip if token matches — not just a soft flag)
- **Email sender + recipient domains** (regex match against `## Domain hard-skips`)
- **Email body first 500 chars** (HARD skip if token matches — catches insurance claim numbers, policy IDs, medical refs)

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
תביעה
תיק תביעה
רפואי
רפואה
בריאות
טיפול רפואי
טיפול נפשי
טיפול פסיכולוגי
טיפול תרופתי
טיפול זוגי
פסיכולוג
פסיכולוגיה
פסיכיאטר
תרופה
מרשם
אבחון
כספים
כספי
כסף פרטי
חשבון בנק
חשבון בנק פרטי
העברה בנקאית
כרטיס אשראי
משכורת
שכר חודשי
מס הכנסה
מסים פרטיים
ביטוח
ביטוח ישיר
ביטוח לאומי
פוליסה
משאבי-אנוש
אישי
פרטי
פרטית
משפחה
משפחתי
ילדים
גירושין
משמורת
סודי
חסוי
זוג
זוגי
זוגיות
בקשות זוגיות
יחסים
ירושה
צו ירושה
אישור ניהול חשבון
אישור ניקוי מס
חשבונית פרטית
חשבונית אישית
משכנתא
פיצויים
תאונה
תאונות
נפגע
תיק תביעות
```

---

## Domain hard-skips (regardless of label)

Any participant in a thread whose email address matches one of these regex patterns → skip the whole thread.

### Hardcoded domain regex (always-skip)

```
# Israeli health funds (kupot cholim) — exact + subdomains
^.*@(clalit|maccabi4u|meuhedet|leumit|maccabi-health)\.(org|co)\.il$
^.*@.*\.(clalit|maccabi|meuhedet|leumit)\.(org|co)\.il$

# Generic medical/legal TLDs and patterns
^.*@.*\.law(\..+)?$
^.*@.*\.med(\..+)?$

# Israeli insurance companies — these often correspond via numeric or obfuscated subdomains
^.*@5555555\.co\.il$                # ביטוח ישיר (Direct Insurance) representative addresses
^.*@bituachyashir\..+$
^.*@(harel|menora|migdal|clal|phoenix|ayalon|shirbit)\.(co\.il|com)$
^.*@.*\.bituach\..+$                # generic "bituach" (insurance) subdomain

# Israeli government — extra caution; skip by default (user can whitelist if they work WITH gov)
^.*@.*\.gov\.il$
^.*@.*\.muni\.il$
^.*@.*\.idf\.il$

# Israeli courts / legal
^.*@.*\.court\.gov\.il$
^.*@.*\.justice\.gov\.il$

# Banks — Israeli (commented; uncomment if you don't work WITH the bank as a client)
# ^.*@(mizrahi-tefahot|hapoalim|leumi|discountbank|fibi|mercantile|igud|union-bank)\..+$
```

### User-configurable bank list

Banks are the tricky case. The deck's intended user (Idan @ 82Labs) has Mizrahi Bank as a *client*, not a private banking relationship — so the entire `mizrahi-tefahot.co.il` domain must NOT be skipped. Other banks the user has private accounts with should be skipped.

Phase 0.4 (orientation questions) asks:

> **HE:** "האם אתה עובד עם בנק כלקוח? אם כן — איזה? אדלג על דומיין הבנקים שאתה לא עובד איתם."
>
> **EN:** "Do you work WITH any banks as clients? If yes, which? I'll skip every bank domain you DON'T work with."

Capture the answer in `_meta/skip-overrides.json` as:
```json
{ "bank_clients": ["mizrahi-tefahot.co.il"], "bank_personal_skip": ["leumi.co.il", "discountbank.co.il"] }
```

The skill then dynamically extends the regex list above with the user's personal-skip banks.

### Subject-line content scan (defense in depth)

Even when domain doesn't match, scan the subject line + first 500 chars of body against the EN+HE token lists above. **Hard skip** (not soft flag) on match.

### Tokens that REQUIRE compound matching (false-positive history)

Some Hebrew words are too broad to skip in their bare form because they have legitimate business uses. Match these ONLY in their compound forms listed in the token list above:

- `בנק` (bank) — bare form is a false-positive trap. `בנק שעות` ("block of hours") is a business idiom. Only skip on `חשבון בנק`, `העברה בנקאית`, `חשבון בנק פרטי`.
- `טיפול` (treatment/handling) — bare form catches IT "treatment", cyber "incident handling", etc. Only skip on `טיפול רפואי`, `טיפול נפשי`, `טיפול פסיכולוגי`, `טיפול תרופתי`, `טיפול זוגי`.
- `חשבונית` (invoice) — bare form catches every business invoice the user sends. Only skip on `חשבונית פרטית`, `חשבונית אישית`, or in combination with `אישור ניהול חשבון`/`אישור ניקוי מס` (both signal personal tax/banking context).
- `כסף` (money), `מס` (tax) — same logic. Bare forms too broad. Only skip on `כסף פרטי`, `מס הכנסה`, `מסים פרטיים`.

This list catches:

- "Re: בהמשך לפנייתך [מס' פניה - ...]" — insurance claim correspondence (via `פניה`/`תביעה`/domain)
- "תיק תביעה מספר: ..." — legal/insurance (via `תיק תביעה`)
- "תשלום בנושא טיפול רפואי" — medical (via `טיפול רפואי`)
- "Re: בקשות זוגיות" — couples therapy (via `בקשות זוגיות`/`זוגיות`)
- "אישור ניהול חשבון פרטי" — personal banking (via `אישור ניהול חשבון`)
- "פוליסה" — insurance policy
- "מס הכנסה — דרישה" — tax demands (via `מס הכנסה`)

Does NOT catch (intentional):
- "חשבונית עבור 300 שעות פיתוח" — business invoice with the word `בנק` as part of `בנק שעות` idiom
- "תשלום בנושא טיפול סייבר" — IT/cyber handling (bare `טיפול` no longer matches; but `סייבר` may signal sensitive incident — review manually via `_meta/skipped.md` rather than auto-skip)

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
