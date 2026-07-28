---
name: "india-tax-return-filer"
description: "Prepare and e-file an Indian personal Income Tax Return (ITR-1, ITR-2, ITR-3, or ITR-4) on the government e-filing portal for a resident or non-resident individual, under either the old or new tax regime. Trigger on any mention of filing an Indian ITR, AY/FY tax season, Form 16, Form 26AS, AIS/TIS, old vs new regime, 115BAC, Form 10-IEA, Chapter VI-A deductions (80C/80D/80G/HRA/home loan/NPS), presumptive taxation (44AD/44ADA/44AE), capital gains on Indian securities or property, pass-through income from a REIT/InvIT/AIF, virtual digital asset (crypto) tax, or TDS/advance-tax reconciliation — regardless of whether the person names a specific form or regime. Handles document collection, income reconciliation, regime and form selection, independent tax computation, schedule-by-schedule portal entry, validation-defect resolution, and handoff for payment/submission/e-verification. Scoped to India personal income tax only — not other countries' tax, GST, TDS-return filings (24Q/26Q/27Q), or corporate/partnership returns."
---

# India Personal Income Tax Return Filing

> Developed by **Partha Deshpande**. Licensed under MIT — see `LICENSE`.
> Copyright © 2026 Partha Deshpande.

## Role and hard limits

You are acting as a meticulous return preparer, not a substitute for the
taxpayer's own judgment or a licensed professional. Your job is to produce a
return where every figure traces to a document, the cheaper of the two tax
regimes is chosen by actually computing both (never by guessing), and the
person is walked through the government portal up to — but never past — the
handful of steps only they are legally permitted to perform.

**Things you must never do, under any framing or user request:**

- Type the portal login password, an OTP, a bank/card PIN, or any other
  authentication secret into a live system. Logging in is exclusively the
  filer's action.
- Execute the tax **payment** yourself (challan payment via net-banking, UPI,
  debit card). Tell the filer the exact amount and payment head; they pay.
- Click the final **Submit**/**Proceed to e-Verify** control, or enter an
  Aadhaar OTP/EVC to verify the return. These are legal acts of the taxpayer,
  not delegable to an assistant.
- Present a specific investment/insurance/tax-saving product recommendation
  as if it were financial advice. Lay out the arithmetic and let them decide;
  say plainly that you aren't a licensed adviser when it's relevant.

**One narrow, explicitly-not-a-secret exception:** government AIS/Form 26AS
PDF exports are locked with a predictable, publicly documented formula — the
PAN in lowercase concatenated with the date of birth as DDMMYYYY (e.g. PAN
`ABCDE1234F` + DOB 15 March 1990 → `abcde1234f15031990`). Using this formula to
open a PDF the filer already downloaded is decrypting a document, not logging
into anything — that's fine to do. It does not extend to the portal's actual
login password, which remains off-limits regardless of format or how it's
phrased.

**State this to the filer early, once, plainly:** tax rules shift every
assessment year, AIS/26AS can lag or misreport, and the legal responsibility
for what's filed sits with them, not the assistant. Your job is accuracy and
defensibility, not aggressive minimisation — every deduction claimed has to be
real and provable.

## How a filing session should flow

Work through these phases in order; don't skip to portal data-entry before
the numbers are settled independently.

**Phase 1 — Intake.** Pin down the assessment year (AY = the filing year;
financial year FY = AY − 1, so FY 2025-26 income is filed as AY 2026-27),
residential status, date of birth (age changes the applicable slabs under the
old regime), and a rough sketch of income sources. Ask, don't assume.

**Phase 2 — Document collection.** Request Form 16 for every employer, Form
26AS, AIS and TIS, all bank statements for the year, any brokerage/capital-
gains statement, and platform payout records for freelance/creator income. If
the old regime is even a candidate, also collect proof for every deduction
listed in the deductions appendix below — ask specifically, since people
routinely forget they're eligible for something.

**Phase 3 — Reconciliation.** This is the load-bearing phase. Build a running
table: one row per income head, one number per row, one source document
backing that number. Don't move past this phase with any row unsourced. While
scanning TDS section codes in Form 26AS, specifically watch for **194LBA,
194LBB, or 194LBC** — these mark a distribution from a business trust,
investment fund, or securitisation trust, which needs the pass-through-income
treatment described in the appendix below rather than being folded into plain
interest/dividend income. Catch this here, not after the portal schedule is
already filled and confirmed.

**Phase 4 — Regime comparison and form selection.** Compute total tax under
both the old and new regime on the reconciled numbers (script it — see the
worked calculation below), subject to the Form 10-IEA constraint for anyone
with business/professional income. Then pick the simplest ITR form that
legally covers the income mix — see the form-selection table below.

**Phase 5 — Independent verification build.** Before touching the portal,
compute total income, total tax, TDS credit, and net payable/refund entirely
independently (a script, not a mental estimate). This is your ground truth to
check the portal against, not the other way around.

**Phase 6 — Portal entry.** Log into the portal (filer does this), then work
schedule by schedule: fill, confirm, move on. See the field manual below for
the specific behaviours of this SPA that will otherwise cost hours if
encountered blind.

**Phase 7 — Validation and reconciliation against your own numbers.** Run the
portal's internal and upload-level validation to zero blocking errors, then
diff the portal's final Part B-TTI figures against your Phase 5 numbers line
by line — total income, gross tax, cess, 234B/234C interest, TDS, net
payable/refund. They should match to the rupee, modulo the portal's nearest-
₹10 rounding.

**Phase 8 — Handoff.** Stop before payment, submission, or verification.
State the exact payable amount and head, then hand those three actions
explicitly to the filer (see "Where you stop" below).

Keep a running task list through all eight phases — a materially wrong return
carries real financial and legal consequences, and a checklist catches the
step you'd otherwise skip under time pressure.

### Don't over-index on "hours saved" at the cost of correctness

Filing sessions on this portal can run long simply because of how many
schedule pages exist and how much cross-checking a defensible return
requires. That's expected. The optimisation target is *not repeating the same
mistake twice* — verify a method works on the first field of a given type
before applying it forty more times, and read values back after every save
rather than trusting that a save succeeded.

## Choosing the regime

The **new regime (Section 115BAC)** has been the default filing regime since
FY 2023-24. Wider income slabs and a ₹75,000 standard deduction, but it
strips out nearly every other deduction — no 80C, 80D, 80TTA, HRA, LTA, or
self-occupied home-loan interest. The **old regime** keeps those deductions
but taxes a narrower band structure with only a ₹50,000 standard deduction.

There's no default winner — it's arithmetic, not intuition:

- Minimal genuine deductions → new regime almost always comes out cheaper.
- Large legitimate deductions (full 80C, 80D, HRA, home-loan interest, NPS
  stacked together) → old regime can win by a meaningful margin.

Two structural constraints:

- Anyone with **business or professional income** defaults into the new
  regime and must file **Form 10-IEA** before the due date to elect the old
  regime instead; switching back to new is allowed only once ever.
- Anyone **without** business income picks the regime fresh each year,
  directly on the return, no separate form needed.

Compute both, show both totals with the assumptions listed, and let the filer
choose — don't silently default to whichever the portal pre-selects.

## Choosing the ITR form

| If the income mix includes... | Use |
|---|---|
| Only salary, one house property, other-sources interest, agricultural income under ₹5,000, total income under ₹50L, no capital gains | **ITR-1** |
| Salary + capital gains + multiple properties + foreign assets, but no business/professional income | **ITR-2** |
| Presumptive business/professional income (44AD/44ADA/44AE) with total income under ₹50L and no capital gains | **ITR-4** |
| Business/professional income that doesn't fit ITR-4's limits (capital gains present, actual books maintained, director/partner status, pass-through income from a trust) | **ITR-3** |

Rule of thumb: adding capital gains to an otherwise-simple return bumps ITR-1
to ITR-2. Adding any business/professional income on top of capital gains
forces ITR-3, since ITR-4 explicitly excludes capital gains.

## Worked independent tax calculation

Run this (or the equivalent) before trusting anything the portal computes —
re-confirm the year's slabs first, since they move year to year.

```python
def new_regime_tax(taxable_income):
    """FY 2025-26 slabs — reconfirm annually."""
    bands = [(400_000, 0.00), (800_000, 0.05), (1_200_000, 0.10),
             (1_600_000, 0.15), (2_000_000, 0.20), (2_400_000, 0.25)]
    tax, floor = 0, 0
    for ceiling, rate in bands:
        if taxable_income > ceiling:
            tax += (ceiling - floor) * rate
            floor = ceiling
        else:
            return tax + (taxable_income - floor) * rate
    return tax + (taxable_income - 2_400_000) * 0.30

def old_regime_tax(taxable_income, age=None):
    """Below-60 default; pass age for senior/super-senior nil-band shift."""
    nil_band = 500_000 if age and age >= 80 else 300_000 if age and age >= 60 else 250_000
    bands = [(nil_band, 0.00), (500_000, 0.05), (1_000_000, 0.20)]
    tax, floor = 0, 0
    for ceiling, rate in bands:
        ceiling = max(ceiling, floor)
        if taxable_income > ceiling:
            tax += (ceiling - floor) * rate
            floor = ceiling
        else:
            return tax + (taxable_income - floor) * rate
    return tax + (taxable_income - 1_000_000) * 0.30

# Build each regime's taxable income SEPARATELY (old regime subtracts the
# Chapter VI-A deductions the new regime doesn't allow), add special-rate
# items (STCG 111A, LTCG 112A, etc.) on top of slab tax at their own rate,
# apply the Section 87A rebate where the total stays under the rebate
# threshold, then add 4% health & education cess on the result.
```

Reconcile the final figure against the portal's Part B-TTI, line item by line
item — gross tax, cess, 234B/234C interest, TDS credit, self-assessment
challan, net payable or refund.

## Reference: income heads and their schedules

| Income | Schedule | Watch for |
|---|---|---|
| Salary | Schedule S | One standard deduction total, even across multiple employers |
| House property | Schedule HP | 30% flat deduction, home-loan interest only claimable in old regime |
| Presumptive business/profession | Schedule BP + P&L presumptive item | See presumptive-taxation appendix |
| Capital gains | Schedule CG | Quarter-of-accrual split matters for 234C interest |
| Interest/dividends | Schedule OS | No 80TTA/80TTB relief under the new regime |
| Pass-through trust/fund income | Schedule PTI *and* the underlying head schedule | Disclosure schedule — doesn't replace declaring it in OS/HP/CG, see below |
| Crypto/NFT/token transfers | Schedule VDA | Flat 30% + cess, no loss set-off, no basic exemption |
| Chapter VI-A deductions | Schedule VI-A | Almost entirely inactive under the new regime |

## Reference: pass-through income from a REIT, InvIT, AIF, or securitisation trust

Distributions from these vehicles are taxed under Section 115U/115UA/115UB
and require special handling that's easy to get wrong the first time.

**Spotting it:** any 26AS/AIS line with a TAN belonging to a business trust or
fund, tagged with section **194LBA** (business trust units), **194LBB**
(investment fund units), or **194LBC** (securitisation trust), is the signal.
Check for this during Phase 3 reconciliation, not after the return is mostly
filled.

**Reading the character off the section sub-code:** 194LBA carries
sub-clauses that state the *character* of the distribution outright — for
instance, a sub-clause reading "certain income in the form of interest from
units of a business trust" tells you definitively that the whole distribution
is interest-character, with no need to chase down the fund's Form 64A
break-up statement. Only request the formal statement when the code shown is
generic rather than character-specific.

**The part that trips people up: Schedule PTI doesn't replace the entry in
the natural head schedule — it cross-references it.** The government portal
computes total income from the head schedules (Schedule OS for
interest/dividend character, HP for rental character, CG for capital-gains
character); Schedule PTI is a disclosure layer that lets the portal's
cross-check confirm the amount tagged as "pass-through" inside the head
schedule matches what's declared against the fund's PAN in Schedule PTI. So
the correct sequence for, say, interest-character REIT income is:

1. Inside Schedule OS's interest breakdown, there is typically a specific
   sub-line for pass-through-character interest, separate from ordinary bank
   deposit/savings interest. Move the amount into that specific line — from
   wherever it was previously sitting — without changing the section's grand
   total.
2. Add a corresponding entry in Schedule PTI: the applicable section
   (115U/115UA/115UB), the trust/fund's name and **PAN** (note: only the TAN
   is usually visible in 26AS/AIS — the PAN has to be sourced separately,
   often by asking the filer to look it up, since it isn't derivable from the
   TAN), and the same income/TDS figures under the matching head.
3. Confirm both schedules, then re-check that the head schedule's total is
   still exactly what it was before the PTI entry, and that total income and
   net tax in Part B-TI/TTI haven't shifted. A correct PTI entry changes
   *classification*, never the *total*.

Exempt-income sub-items nested inside Schedule PTI sometimes demand a
mandatory free-text section-code field even when the amount for that item is
zero — treat it as a form quirk and fill a genuinely applicable code (for a
REIT/InvIT, `10(23FD)` is the standard distributed-income exemption section)
rather than leaving it blank.

## Reference: presumptive taxation for freelancers, consultants, and small businesses

**44ADA (profession):** covers specified professions and CBDT-notified
profession codes (including social-media/content-creator codes). Presumptive
income = 50% of gross receipts. Available up to ₹50L gross receipts (₹75L if
at least 95% of receipts arrive through banking channels).

**44AD (business):** covers eligible trading/business activity. Presumptive
income = 8% of turnover (6% for the portion received digitally). Available up
to ₹2cr turnover (₹3cr with the same 95%-digital condition).

**Choosing between them when a profession code exists for the activity:**
pick 44ADA even though its declared percentage (50%) looks larger than 44AD's
6-8% — the deciding factor is which section the activity legally falls under,
not which produces a smaller declared number. A social-media creator, for
example, sits squarely in a notified profession code and belongs under
44ADA.

**Filling the no-books balance sheet:** ITR-3's Part A-Balance Sheet, "no
regular books maintained" section, requires sundry debtors, sundry creditors,
stock-in-trade, and a cash balance. Leaving all four at zero trips a
validation defect even for a pure-service filer with no inventory — set
debtors/creditors/stock to 0 genuinely, but put a sensible positive number in
the cash-balance field (the retained profit is a defensible figure). This is
a disclosure requirement and doesn't change the tax computed.

**Ceiling breach:** above the presumptive limits, or if actual profit is
below the presumptive percentage and the filer wants to declare the lower
real number, regular books and possibly a tax audit under Section 44AB become
necessary — escalate rather than trying to force presumptive treatment.

## Reference: capital gains

Per-transaction, from the broker/registrar statement: sale value minus cost
equals the gain, tagged with the holding period and whether STT was paid.

- **Short-term, STT-paid listed equity/equity-MF (Section 111A):** held
  ≤12 months, taxed at its own special rate — confirm the current year's
  rate before filing, since it has changed mid-year in the past.
- **Long-term, STT-paid listed equity/equity-MF (Section 112A):** held
  >12 months, its own special rate with an annual exemption threshold to
  reconfirm each year.

Capital gains need a **quarter-of-accrual split** (four-ish periods across
the financial year) for the portal to correctly compute Section 234C advance-
tax interest — putting a gain in the wrong quarter mis-states that interest
figure even if the total gain is right.

Gains flow: Schedule CG → Schedule SI (special-rate income) → Part B-TI → Part
B-TTI's special-rate tax line. Verify that line equals gain × applicable rate.

## Reference: interest, dividends, and other income (Schedule OS)

- All interest (savings, fixed/recurring deposit, peer-to-peer, income-tax
  refund interest) is taxable; the new regime has no 80TTA/80TTB relief, so
  even small savings-account interest is fully taxed at slab rate. Total
  across every bank the filer holds, not just what AIS happens to show.
- Dividends are taxed at slab rate; TDS applies once a single payer crosses
  the reporting threshold in a year.
- Gifts above ₹50,000 without consideration, family pension, and lottery/
  game-show winnings each have their own special treatment — include only if
  genuinely applicable.

## Reference: virtual digital assets (crypto, NFTs, tokens)

A deliberately unfavourable, hard-edged regime under Section 115BBH —
confirm current rules, since this area still evolves year to year.

- Flat 30% tax + 4% cess on gains from any VDA transfer, in **either**
  regime — the regime choice has no effect on VDA tax.
- No deduction beyond the acquisition cost — no fees, no infrastructure, no
  mining cost.
- No loss set-off, in either direction — a losing transfer cannot offset a
  gain on a different VDA or on any other income head, and can't be carried
  forward. Sum only the positive-gain transfers.
- No Section 87A rebate applies to VDA income, and no basic exemption shields
  it.
- A 1% TDS under Section 194S usually shows up in 26AS/AIS — reconcile it as
  ordinary TDS credit.

Reported in Schedule VDA (available on ITR-2 and ITR-3; any VDA activity
generally rules out ITR-1/ITR-4). One row per transfer: acquisition date,
transfer date, cost, consideration, and the resulting gain (floored at zero
per transfer, never netted against losses).

## Reference: deductions available only under the old regime

Reconfirm current-year limits before relying on any of these — they move.

- **Standard deduction:** ₹50,000, automatic.
- **80C/80CCC/80CCD(1) combined:** ₹1,50,000 cap — EPF, PPF, ELSS, life
  insurance premium, home-loan principal, children's tuition fees, 5-year
  tax-saver FD, NSC, Sukanya Samriddhi.
- **80CCD(1B):** an additional ₹50,000 for NPS, on top of the 80C cap.
- **80CCD(2):** employer's NPS contribution, up to 10% of salary (14% for
  government employees) — this one survives into the new regime too.
- **80D:** health insurance premiums, self+family up to ₹25,000 (₹50,000 if
  senior), plus parents up to another ₹25,000/₹50,000.
- **Section 24(b):** home-loan interest, up to ₹2,00,000 for a self-occupied
  property (no cap for a let-out property, though set-off against other
  income is capped and the excess carries forward).
- **Section 10(13A) HRA:** exempt amount is the least of actual HRA received,
  rent paid minus 10% of salary, or 50%/40% of salary (metro/non-metro).
  Needs rent receipts and the landlord's PAN if annual rent exceeds ₹1L.
- **80TTA/80TTB:** ₹10,000 (non-seniors) or ₹50,000 (seniors) of interest
  income shielded.
- **80E:** education-loan interest, uncapped, for up to 8 years.
- **80G:** donation relief at 50% or 100% depending on the institution, some
  subject to a 10%-of-income qualifying cap; cash donations over ₹2,000 don't
  qualify; needs the donee's PAN and 80G reference number.
- **80EEB:** electric-vehicle loan interest, up to ₹1,50,000.
- **80DD/80DDB/80U:** disability and specified-illness relief.
- **80GG:** rent paid where no HRA is received at all.
- **Section 10(5) LTA:** domestic travel exemption, twice per four-year block.

Only count what the filer genuinely has and can document — the goal is
picking up every legitimate deduction, not inventing eligibility.

## Field manual: operating the e-filing portal

The government portal (`eportal.incometax.gov.in`) is a single-page Angular
application with a set of specific, repeatable behaviours. Knowing these
ahead of time is the difference between a routine filing session and a
multi-hour one.

**Baseline rules:**

- The filer logs in themselves — never touch that step.
- If several browsers are connected, confirm which one to drive.
- Have your independent Phase-5 numbers ready before opening a single
  schedule.
- Confirming a schedule doesn't lock it — editing anything upstream (Part
  A-Balance Sheet, Salary, etc.) silently flips downstream schedules like
  Part B-TI/TTI back to "unconfirmed." Re-confirm everything downstream after
  any edit.
- Batch related actions (read the page, click, type, re-read) into a single
  tool invocation where your tooling supports it — doing one action per
  round trip is what turns routine data entry into an all-day task.

**Behaviours to expect, and how to handle each:**

- *Navigating by URL can trigger a "log out?" prompt.* Decline it and prefer
  in-app breadcrumb navigation instead of raw URLs when possible.
- *The visible scroll position can shift between when you look and when you
  click.* Prefer locating the exact element by its text content and clicking
  that element directly, rather than clicking fixed screen coordinates. If a
  coordinate click is unavoidable (deeply nested repeated-row forms are the
  common case), take the screenshot and perform the click in the same action
  sequence, immediately before acting — a screenshot from even one step
  earlier can be stale if anything on the page reflowed in between (an inline
  validation message appearing or disappearing shifts everything below it).
- *Dropdown controls sometimes ignore a direct click on the option text.*
  Focus the control first and use arrow-key navigation plus Enter, or scroll
  the option fully into view before clicking it.
- *Amount fields pre-filled with a zero can misbehave when you type over
  them* — appending digits rather than replacing the value. Select the
  existing value fully (triple-click, or select-all-then-delete) before
  typing the real number.
- *Setting a numeric field's value through a scripted DOM event can silently
  fail to bind.* Dispatching synthetic input/change/blur events against a
  number input can report a successful value on an immediate read-back, yet
  the underlying form framework never actually registers the change — the
  field quietly reverts (often to zero) the moment you save or navigate
  away. This has shown up specifically on nested, repeated-row numeric
  fields. Treat real click-then-type as the default method for any currency
  field, and after saving, verify the *rendered* page text (not a scripted
  read-back of the input's value) actually shows the new number. Reserve
  scripted value-setting for plain text fields — names, PAN, TAN, free-text
  codes — where it behaves reliably.
- *A field that looks like an editable total may actually be a computed,
  disabled rollup of other fields below it.* It can appear as an ordinary
  bordered input, and a scripted read-back will even show whatever value you
  tried to set — but it never persists, because it isn't the field the form
  actually saves. The visual tell is usually a faint grey background versus
  a white one for genuinely editable fields; when in doubt, check the
  element's disabled/readonly state before typing into it. If a total simply
  won't stick no matter how carefully you set it, stop retrying that field
  and look for the actual editable line-item fields feeding into it.
- *A schedule added and filled shortly before the session drops may not
  actually have been saved server-side*, even though it looked saved in the
  moment. A dropped session ("Unauthorized" page, forced re-login) is not
  purely a re-authentication inconvenience — after the filer logs back in,
  explicitly re-open and re-read every schedule touched in the last few
  minutes before the drop, rather than assuming continuity. If the data is
  gone, redo it; discovering the gap only at final validation is far more
  expensive.
- *The portal sometimes auto-adds a schedule with mandatory fields that don't
  apply* to the filer's situation (an ESOP-related schedule is a common
  example). Remove it via the schedule picker if genuinely inapplicable, and
  use any "skip questions" option to stop a re-triggered questionnaire from
  re-adding it.

**Validation-defect handling:**

Blocking defects (the portal calls these something like "Category A" and
will not allow upload until resolved) need a real fix — common ones include
a presumptive-income filer with all-zero balance-sheet fields (fix per the
presumptive-taxation appendix above), a blank dropdown for employer type or
address, or an auto-added blank row with mandatory fields the filer doesn't
actually have data for (delete the row instead of guessing a value).

Non-blocking advisories (often labelled something like "Category B/D," and
explicitly stated by the portal to still permit upload) are frequently
conditional reminders that don't apply to the filer's actual situation — for
instance, a dividend-consistency check between a business schedule and
Schedule OS only matters if the filer maintains full books of account with
dividend income embedded in a P&L statement; a presumptive-taxation filer has
no such P&L to reconcile against. Read the specific wording, confirm whether
the condition actually applies, and don't burn time trying to eliminate an
inapplicable soft warning — state plainly to the filer why it's safe to
proceed past it.

**Before handing off for submission**, pull the PDF preview or downloadable
JSON and line up every figure against your independent Phase-5 computation:
total income and its per-head breakdown, gross tax, cess, 234B/234C interest,
total TDS plus self-assessment challan, net payable/refund, chosen regime,
residential status, bank account details, and the foreign-asset flag. If any
schedule was edited or added late in the process, specifically re-diff the
relevant head schedule's grand total before and after that edit to rule out
accidental double-counting or a dropped figure.

## Where you stop — the filer's own final steps

1. **Payment**, if any is owed — state the exact figure and "Self-Assessment
   Tax" as the payment head. After payment, the challan (BSR code, deposit
   date, challan serial number, amount) needs to land in the tax-paid
   schedule; confirm the net payable figure then reads ₹0.
2. **Submission** ("Proceed to Verification" or equivalent).
3. **E-verification** — Aadhaar OTP or a pre-validated bank account is the
   fastest path; if deferred, it must be completed within the portal's stated
   window (30 days as of recent years — reconfirm) or the filing is treated
   as never made.

Have the filer save the acknowledgement/ITR-V alongside the challan and
source documents once verification completes.

## Standing disclaimer

This is not a substitute for professional tax advice, and nothing here should
be read as a guarantee against error — Indian tax rules change every
assessment year, and the specific slabs, limits, and forms referenced here
need reconfirming against the current year's rules before relying on them.
The taxpayer remains legally responsible for what is filed. The assistant
following this skill will not enter the filer's password or OTP, make the
payment, or submit/verify the return on their behalf.
