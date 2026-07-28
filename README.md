# india-tax-return-filer

A Claude/Cowork skill for preparing and e-filing an Indian personal Income
Tax Return (ITR-1, ITR-2, ITR-3, or ITR-4) on the government e-filing portal
(`eportal.incometax.gov.in`), under either the old or new tax regime.

Developed by **Partha Deshpande**.

## What it does

- Walks through document collection (Form 16, Form 26AS, AIS/TIS, bank and
  broker statements) and reconciles every income figure to a source document.
- Computes tax under both the old and new regime independently, so the
  cheaper regime is chosen by arithmetic rather than guesswork.
- Picks the correct ITR form based on the actual income mix.
- Drives the government e-filing portal schedule-by-schedule, with a field
  manual covering the portal's specific quirks (dropdown behavior, numeric
  field persistence, session-drop data loss, validation-defect handling).
- Handles pass-through income from REITs/InvITs/AIFs (Schedule PTI),
  presumptive taxation for freelancers/creators (44AD/44ADA), capital gains,
  and virtual digital asset (crypto) taxation.
- Stops before payment, submission, and e-verification — those steps always
  remain the taxpayer's own action.

## What it will never do

- Enter your portal login password, OTP, or any other authentication secret.
- Make the tax payment on your behalf.
- Click Submit / Proceed to e-Verify, or enter an Aadhaar OTP/EVC.
- Give you a specific "buy this policy" investment recommendation as
  financial advice.

## Status

The general tax logic (regime comparison, deductions, form selection) applies
uniformly across all four forms. The government e-filing portal's
form-specific automation quirks have been most thoroughly exercised on
**ITR-3** to date; ITR-1/2/4 rely on the same general portal-mechanics
playbook but haven't each been independently field-verified yet. Contributions
and reports from real filings on other forms are welcome.

## How to use it

Drop `SKILL.md` into your Claude Code / Claude Agent SDK / Cowork skills
directory, or install the packaged `.skill` file directly if you're using
Cowork.

## Disclaimer

This is not a substitute for professional tax advice. Indian tax rules change
every assessment year — always reconfirm current-year slabs, limits, and
forms before relying on any figure here. The taxpayer remains legally
responsible for what is filed.

## License

MIT — see [LICENSE](./LICENSE). Copyright © 2026 Partha Deshpande.
