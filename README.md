# CashSync
# CashSync — AI Finance Controller / Automated Reconciliation Agent

## Problem

Merchants maintain an internal ledger of transactions, but the bank's record of what actually settled rarely matches it perfectly. Common causes of mismatch include:

- Settlement delays (a transaction posts a day or two later)
- Fee deductions (bank shows a smaller net amount than the ledger's gross amount)
- Corrupted or truncated transaction references
- Duplicate bank entries
- Bank-only entries (e.g. refunds) with no corresponding ledger record

Manually reconciling these two sides is slow and error-prone. This project builds an agent that automates the matching process, flags genuine exceptions for human review, and reports its own accuracy — closing one finance-ops loop end to end.

## Approach

1. **Synthetic ground truth**: generate 60 clean ledger transactions with known transaction IDs, amounts, and dates.
2. **Deliberately corrupted bank statement**: derive a "bank statement" from the ledger, introducing realistic discrepancies — missing entries, fee deductions, settlement delays, corrupted references, duplicate rows, and bank-only entries — so the ground truth is known but the matching problem is genuinely messy.
3. **Tiered matching engine**:
   - **Tier 1 (exact)**: transaction ID, amount, and date all match exactly.
   - **Tier 2 (tolerance)**: same transaction ID, with amount within a configurable tolerance (e.g. fee deduction) and date within a configurable window (e.g. settlement delay).
   - **Tier 3 (fuzzy)**: no exact ID match, but a high string-similarity match (via `rapidfuzz`) combined with close amount and date.
4. **Exception handling**: anything left unmatched is placed in an exceptions table with a specific reason — pending settlement, unrecorded bank-only transaction, or possible duplicate bank entry.
5. **Audit trail**: every match records its tier, confidence score, amount/date differences, and a human-readable reason — so every decision the agent makes is explainable.

## Results

- **Match rate**: 59/60 = 98.33%
- **Tier breakdown**: 45 exact, 13 tolerance, 1 fuzzy
- **Exceptions**: 7 transactions flagged for human review (1 pending settlement, 3 likely duplicate bank entries, 3 bank-only/unrecorded entries)

All matched pairs and exceptions include a reason, so the results are auditable rather than a black box.

## Files

| File | Description |
|---|---|
| `AI_Finance_Controller.ipynb` | Full notebook: data generation, matching engine, results |
| `ledger.csv` | Synthetic merchant ledger (ground truth) |
| `bank_statement.csv` | Synthetic bank statement (corrupted version of the ledger) |
| `matched.csv` | All matched transaction pairs with tier, confidence, and reason |
| `unmatched_ledger.csv` | Ledger transactions with no bank match |
| `unmatched_bank.csv` | Bank transactions with no ledger match |
| `exceptions.csv` | Final exception list with categorized reasons |

## How to run

1. Open `AI_Finance_Controller.ipynb` in Google Colab or Jupyter.
2. Run all cells in order — the notebook generates its own synthetic data, so no external files are required.
3. Review the printed match rate, tier breakdown, and exceptions table at the end.

## Next steps

- **Interactive dashboard**: build a Streamlit app to browse matched transactions and exceptions visually, with summary metrics at a glance.
- **Configurable thresholds**: expose amount tolerance, date window, and fuzzy-match threshold as adjustable parameters for different merchant risk profiles.
- **Richer exception categorization**: extend reason codes to cover more real-world cases (e.g. partial refunds, multi-currency conversion differences, tax-line splits).
- **Live data connector**: replace synthetic CSVs with a real reconciliation source (e.g. a payment gateway settlement report) for a production-ready version.
