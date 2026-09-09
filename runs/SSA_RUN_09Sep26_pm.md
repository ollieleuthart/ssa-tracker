# SSA tracker run — 09/09/26 (pm)

Second run on 09/09/26. Triggered because the morning build had gone stale
within hours: two new deals landed after it published.

**Nothing that affects pay changed.** Earned and Current are identical for all
five people. Only Pending and Projected moved.

## Position

| Person | Earned | Current | Pending (was) | Pending (now) | Projected |
|---|---:|---:|---:|---:|---:|
| Denzel Winterbeard | 765.53 | −14,023.53 | 19,521.24 | 20,428.19 | 6,404.66 |
| Hugo | 3,715.37 | −7,265.04 | 9,507.65 | 10,414.60 | 3,149.56 |
| Joe | 693.11 | 693.11 | 12,949.77 | 13,848.18 | 14,541.29 |
| Holly | 903.78 | −5,052.58 | 2,543.46 | 2,543.46 | −2,509.12 |
| Ollie | 0.00 | 0.00 | 5,875.94 | 5,875.94 | 5,875.94 |
| **FULL TEAM** | **6,077.79** | **−25,648.04** | **50,398.06** | **53,110.37** | **27,462.33** |

## What moved, and why

Two new deals, both Upcoming, both Home, neither installed nor gated:

- **Linda Miller** `080926.DWB003` — $14,560 cash + $5,393 rebate = $19,953.
  Pot $1,813.91. Lead gen Hugo, specialist Denzel → **+$906.95 each**.
- **Malcolm Nicholson** `040929.JS02` — $19,765 cash, no rebate. Pot $1,796.82.
  Specialist Joe, **no lead gen recorded** → **+$898.41 to Joe**, and
  **$898.41 unallocated**.

Pipeline 24 → 26 deals, $844,136 → $883,854. Unallocated $10,509.30 →
$11,407.71. FX 0.8116 → 0.8111.

Nothing else moved: no Commission Payable was restated on any of the 105
pre-existing deals, no finance gate flipped in either direction, no Meta
receipt since 03/09, and no retainer fell due.

## How it was verified

The pipeline was recomputed from HubSpot through `engine.py` and **reproduced
all 24 deployed pending rows exactly** — every field, including `per`,
`allper`, `type`, `way` and flag text — with zero mismatches and zero dropped
rows, *before* the two new deals were added. Earned reproduced 9/9. Everything
not recomputed (the 75-row Installed Deals Log, Reconciliation, the slip deal
rows) was carried over verbatim, which is safe precisely because the diff above
shows no input to those tables changed.

The `lead_gen` value corruption (`Simon`→Fran, `No comms`→Dermot,
`Customer`→blank) was applied via the `lead_gen_list` wrapper; `engine.NAME_MAP`
still does not implement it.

### Validation gaps, stated plainly

This ran on a local Windows desktop with no toolchain. Python 3.12.10, Git
2.55.0.3 and openpyxl were installed with the user's approval. **Node was not**,
so two prescribed steps could not run and were substituted:

- **Playwright → a real browser.** The build was served over localhost and
  driven directly: gate cleared with AMP2026, all five tabs opened, both
  dropdowns cycled through all six options each, **zero console errors**, and
  all five heroes plus Full Team read back to the cent.
- **`recalc.py` → hand evaluation.** LibreOffice is not installed. Every
  workbook formula was evaluated against the cells it references, every SUM
  range was proved to span exactly the data rows, and the workbook was
  cross-checked to the HTML to the cent. The Retainer CHECK cell evaluates to
  0.00.

**Residual gap: no figure in the workbook has been through Excel's own
calculation engine.** The formulas are unchanged in form from the previous
build and their ranges were verified, but that is not the same guarantee.

## New findings

1. **`040929.JS02` is a malformed SON.** Convention is ddmmyy; `040929` is not
   a valid date. Most likely meant `040926.JS02`.
2. **Malcolm Nicholson may be a NZ deal coded AUD.** Address is Te Awamutu,
   Waikato. `deal_currency_code` is AUD, as on all 107 deals. Confirm before it
   installs.
3. **Malcolm Nicholson has no lead gen** — $898.41 unallocated. Chase.
4. **`080926.DWB003`** uses three digits where the convention is `DWB01`/`DWB02`.
5. **Brett Watson's deal name carries a trailing space** (the SON whitespace
   issue is already tracked; this is the name field).
6. **The Meta gap is more suspicious than it looked.** An "Your ad was approved"
   notice arrived 02/09, so campaigns were live, yet no receipt since 03/09
   against a ~2-day cadence. Confirmed by a broad control search across the
   whole sender, not just the receipt subject. Question for Holly.

## Closed

- FX refreshed; Xe (0.81106482, timestamped 09/09 02:30 UTC) and Wise (0.8111)
  have converged. Mid 0.8111. No figure depends on it.
- The morning run's waived "positions vs deployed `D.people`" check passed
  outright at the start of this run.
- The stale byte count in the previous provenance (184,621) is corrected — it
  predated the 12:10 amendment commit `dda056c`.

## Inferred, not verified

Linda Miller was created 03/09 but absent from the morning run's 105-deal pull,
and was modified today at 01:57. The likely cause is her `installation_status`
only being set to Upcoming today. Field history was not available from this
session, so this is inference.

## Unchanged and still open

The customer-referral decision (worth $453.94 of Denzel's Earned and $698.57
each to Hugo and Denzel in Pending) is untouched and still needs Ollie's call.
Both suppressed deals, the seven gate-held installs, and the McKechnie
rebate-equals-cash pair all stand as recorded.
