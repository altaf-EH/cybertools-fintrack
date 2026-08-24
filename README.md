# CyberTools FinTrack
### Financial Fraud & Mule Account Link Analyzer

Same engine style as **CyberTools CDR Analyzer**, retargeted from call
records to money-transfer records. Point it at any transaction
statement and it profiles every account, flags likely mule accounts,
and maps the money flow between accounts.

## Usage

```
python fintrack_analyzer.py "C:\Users\you\Downloads\statement.csv"
```

or just run it with no arguments and paste/drag the file path when
prompted:

```
python fintrack_analyzer.py
```

The file can be anywhere on the PC — CSV, TSV, or Excel (.xlsx/.xls).

## Input format

The tool auto-detects columns from almost any bank/PSP export. At
minimum it needs:

- a **sender / payer / debit account** column
- a **receiver / payee / credit account** column
- an **amount** column
- a date+time (or single datetime) column

Dozens of common header spellings are recognized automatically
(`From A/C`, `Payer Account`, `Debited Account`, `Sender UPI ID`,
`Amount (Rs)`, `Txn Date`, `UTR`, etc.) — see `COLUMN_ALIASES` in the
script if you want to add more for a specific bank's export format.
Optional columns (transaction type, channel, bank name, remarks,
reference/UTR) are picked up automatically when present and simply
skipped when absent — the tool never fails just because a column is
missing, only if it truly cannot find the three required ones.

## What it detects

For every account seen (as sender or receiver) it computes:

- transaction volume & velocity, unique counterparties
- **pass-through ratio** — how much of the money that comes in goes
  straight back out (the core mule-account signature)
- **same-day in/out** activity (rapid layering)
- **funnel pattern** — many small credits from many distinct senders
  followed by very few, large debits (classic smurfing/mule funnel)
- round-figure transactions (a common structuring indicator)
- odd-hour activity (11 PM – 5 AM)

Each account gets a 0–100 risk score and a HIGH / MEDIUM / LOW rating
with a plain-language list of reasons. A "Top Account Pairs" section
does link analysis — the biggest money-flow relationships between
accounts, useful for building a network graph in a case file.

## Output formats

Every run produces all of the following in `reports/`, so the same
analysis is ready for whichever format an investigator, bank, or
agency asks for:

| File | Purpose |
|---|---|
| `*.pdf` | The formatted CyberTools-branded report (what you'd hand to an officer/committee) |
| `*.xlsx` | Multi-sheet workbook — Summary, Suspicious Accounts, Account Pairs, and every raw transaction row (for evidence/verification, sortable & filterable) |
| `*.txt` | Plain-text version of the same report |
| `*flagged_accounts*.csv` | Flat CSV of only the MEDIUM/HIGH risk accounts, for quick bulk import into a bank's own case-management or blocklist system |
| `*.json` | Full structured report, for handing off to another tool/system programmatically |

## Setup

```
pip install -r requirements.txt
```

Requires the `assets/cybertools_logo1.png` file to sit next to the
script (already included) for the PDF header logo.

## Notes

- All amounts in the report are labeled `Rs.` (kept as text rather
  than the ₹ glyph so it renders correctly in every PDF viewer/font).
- Risk thresholds and scoring weights live in `calculate_risk()` —
  tune them if your definition of "suspicious" differs.
- This is a defensive/investigative triage tool. It surfaces
  candidates for a human analyst to review — it does not make legal
  determinations of fraud or money laundering on its own.
