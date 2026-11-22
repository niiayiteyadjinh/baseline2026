# KG Baseline Completion Monitor

A single-file monitoring dashboard for the KG baseline. It tracks **completion only** — how many
selected schools have submitted a baseline form, by region, district and school. It does not analyse
observation scores.

**Open:** `KG_Baseline_Completion_Monitor.html` (double-click; no server or install needed).

---

## What is on each tab

| Tab | Contents |
|---|---|
| **Overview** | Overall completion KPI, districts and regions started, responses received. **16 region cards**, each with its own colour, showing % completed, schools completed, districts complete and a status badge. Latest submissions and an "attention needed" panel. |
| **Districts** | **261 district cards**, coloured to match their region, showing % completed, schools completed, submissions and status. Filter by region / status, search, sort, or show only districts with submissions. **Click any card** for a detail panel: every selected school with its submission status, and every submission received (date, school as submitted, observer, match quality). |
| **Schools** | The full 1,601-school register with status, submission count and last response. Filter by region/district/status, search, sort, paginate, and click a row for its submissions. |
| **Data quality** | The counting method, unmatched submissions, fuzzy name matches to verify, duplicates, form gaps and links to both Google Sheets. |

Region cards also open a panel listing their districts; district panels open schools; `Esc` or the
backdrop closes any panel.

## How completion is counted

- **Denominator** — 1,601 selected schools across 261 districts and 16 regions (selected-schools sheet).
- **Numerator** — **unique selected schools with at least one baseline submission.** Duplicate
  submissions for the same school count once, so completion cannot exceed 100%.
- **Status bands** — On track ≥75% · Moderate 50–74% · Behind 1–49% · Not started 0%.
- **Name matching** — the response form captures no school code, so names are matched to the selected
  list within the respondent's region and district, using token-level fuzzy matching, containment and
  district anchoring. Matches are graded **Strong** (counted), **Probable/Review** (counted, flagged for
  verification) or **Unmatched** (excluded, shown in Data quality).

  The rules, in order of precedence:

  1. **Exact name** — normalised equality (case, punctuation, `M/A` vs `MA`, `R/C` vs `RC` ignored) wins
     outright. If two selected schools share the same name, the row is counted but listed for
     verification, because only the school code can tell them apart.
  2. **Strong similarity** — ≥95% with a clear lead over the next candidate, or ≥88% with a 10-point lead.
  3. **Probable** — 72–95% with a clear lead; counted, shown in *Name matches to verify*.
  4. **Only school of that name in the district** — for short names such as `Methodist B` or `MA C/D`
     that no similarity score can confirm. If exactly one school in the district the observer selected
     contains every word they typed, it is counted as Probable. Never applied when an exact match exists.
  5. **Wrong district** — if a row is otherwise unmatched but the school exists elsewhere in the same
     region, Data quality names that district so the observer can re-submit correctly.
  6. **School type conflict** — `Presbyterian` against `Islamic` (or any other differing type word) is
     capped, so two different schools with similar names are not merged.
  7. **Unmatched** — excluded from completion and listed with a recommended action.

## Data refresh

- On open, the dashboard pulls **live baseline responses** from the Google Sheets JSON endpoint and
  re-syncs every 5 minutes (and when you return to the tab). The header pill shows `Live`,
  `Offline snapshot` or `Syncing`; the refresh button forces a sync.
- The **selected-schools master is embedded** from the last build. The sheet's public endpoint currently
  exposes only 99 visible rows (the rest of the sheet is filtered/hidden), so it cannot supply the full
  1,601-school denominator. If the sheet is later unfiltered, the dashboard will use it automatically.
- **Header row deleted?** If someone removes the first row of the response sheet, every column arrives as
  "Column 1, Column 2…". The dashboard detects this and works out each column from its contents
  (region, district, the per-region school columns, the free-text school field), so it keeps counting.
- If Google Sheets cannot be reached (no internet, restricted preview, permissions), the page falls back
  to the embedded snapshot and keeps working fully.

To rebuild after changing the selected-schools list:

```bash
python3 src/snapshot.py      # rebuild the embedded fallback from the CSVs in data/
python3 src/assemble.py      # reassemble KG_Baseline_Completion_Monitor.html
```

Keep both sheets shared as "anyone with the link can view" for live responses.

## Data issues worth fixing at source

1. **No school code on the response form.** This is the main source of ambiguity; even with fuzzy
   matching, generic names ("Methodist B", "Anglican A Basic") cannot be confirmed. Add a
   district-filtered dropdown of school codes.
2. **Duplicate submissions** — several schools were submitted 2–4 times. A uniqueness check or a
   note in the form guidance would avoid this.
3. **District not recorded** on a few responses (the district column is region-specific, so respondents
   can submit without selecting one). A single cascading district field would close the gap.
4. One response names a school that is not in the selected list at all (see Data quality).

Source sheets:
- Selected schools — `1oQR0oxCNjfi103l6Rk1nDyoJj-19xIPyNSm261xeQiQ`
- Baseline responses — `186iyjgJmP3UqA1QhL_VlasTeVQg6OP0zd-qZSrx22Wo`

**Hosting:** see `HOSTING_GUIDE.md` for publishing this on Google Sites (host the file, then embed the URL).
