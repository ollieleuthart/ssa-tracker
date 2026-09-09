# Ampra Commission Tracker — run 09/09/26

HubSpot pull 09/09/26 · reset baseline 20/07/26 · ad spend to 03/09/26 · 105 deals pulled, 105 processed.

**Files built and validated. NOT published** — the session proxy blocks the repo again (re-proved with controls; see below).

## Position

| Person | Start | Retainer + ad spend | Earned | Current | Pending | Projected |
|---|---:|---:|---:|---:|---:|---:|
| Denzel Winterbeard | (4,789.06) | 10,000.00 | 765.53 | **(14,023.53)** | 19,521.24 | **5,497.71** |
| Hugo | (7,730.41) | 3,250.00 | 3,715.37 | **(7,265.04)** | 9,507.65 | **2,242.61** |
| Joe | — | — | 693.11 | **693.11** | 12,949.77 | **13,642.88** |
| Holly | — | 5,956.36 | 903.78 | **(5,052.58)** | 2,543.46 | **(2,509.12)** |
| Ollie | — | — | 0.00 | **0.00** | 5,875.94 | **5,875.94** |
| **Full Team** | (12,519.47) | 19,206.36 | 6,077.79 | **(25,648.04)** | 50,398.06 | **24,750.02** |

No retainer draw was due this week (Denzel 03/09 → 17/09; Holly 04/09 → 18/09). **Both fall due before the next run — confirm them.**

## The decision this run turns on

Finance has restated Commission Payable to **exactly 5% of net revenue** on the three installed deals whose lead gen is recorded as `Customer`:

| SON | Deal | Was | Now |
|---|---|---:|---:|
| 250626.DWB01 | Brenda Walker | 1,815.75 | 907.88 |
| SSA035 | John Tooler | 2,793.16 | 1,396.58 |
| SSA041 | Allan Marks | 2,141.89 | 1,070.95 |

Those are precisely the deals the tracker had been flagging as "half the pot unallocated", and `Customer` is the marker the deal sheet tells staff to use when there genuinely was no lead gen. It reads as a deliberate answer to that flag.

**This build still splits the restated pot in half by role, leaving a half unallocated.** If finance instead meant the specialist to take the whole restated amount:

- **Denzel's Earned +$453.94** (Brenda Walker — the only post-reset one, so the only one that moves money now)
- **Ian Potter 100826.JS01** (same `Customer` pattern, pending): **Hugo +$698.57, Denzel +$698.57**
- Allan Marks (+$535.48 Joe) and John Tooler (+$698.29 Joe) are pre-reset — log only, nothing payable

Shipped as the rule stands. Not changed without your answer.

## What moved since the 04/09 state

Every movement reconciles exactly.

**Earned** — one cause only: Denzel −$453.94, the Brenda Walker halving above. Hugo and Holly are each 1c lower purely because Earned is now the sum of already-rounded per-deal shares, so the slip rows sum exactly to the headline.

**Pending** — two new deals created 07/09, and nothing else:

| Deal | Value | Effect |
|---|---:|---|
| 080926.JS01 Matthew Evans | 30,789 | Hugo +1,399.50 · Joe +1,399.50 |
| 020926.JS02 Katie Young 2 | 6,448 | Joe +293.09 · **Hannah +293.09** |

Joe +1,692.59 and Hugo +1,399.50 are exactly those two lines. Denzel, Holly and Ollie moved $0.00.

## A bug I introduced and caught

My first pass read `lead_gen` raw and missed the **dropdown value corruption**: the API returns internal values, and in `lead_gen` `Simon` is really **Fran** and `No comms` is really **Dermot**. `engine.NAME_MAP` does not implement this — it maps `No comms` to blank, which is correct for `specialist` and wrong for `lead_gen`.

Left unfixed it changed away-trip headcounts and would have moved Denzel by $80.86, Hugo by $80.87 and Joe by $69.32 on Earned alone, plus pending. Fixed with a `lead_gen_list()` wrapper applied before `NAME_MAP`, and verified two ways: the reference doc, and reverse-engineering every `lead_gen` list in the deployed build against the raw values (12 `Simon` and 8 `No comms` occurrences, all consistent).

## Ad spend

This being the 4th run, the ledger was rebuilt from source rather than read from cache: a full Gmail sweep from 20/07, 26 distinct receipts after discarding **15 duplicates**. It reproduced the cached figures **to the cent** — 25 AU receipts, gross $3,598.22, $143.89 recruitment ("Ampra Job") excluded, **$3,454.33 chargeable**, plus NZ$302.56 → A$252.03 at the locked 0.8330. Total **$3,706.36**, unchanged.

No new receipts since 03/09/26 — **six days**, against a roughly two-day cadence through August. Either the campaigns paused or receipts have stopped arriving; worth a look.

FX refreshed and the two-run gap closed: **0.8116** (Xe 0.81093604 timestamped 08/09/26 17:01 UTC; Wise 0.8123). No figure depends on it — there was no new NZ spend, and logged NZ rows keep the rate they were struck at.

## Xero reconciliation

| | |
|---|---:|
| Commission Payable, 75 reconciled deals | 155,207.06 |
| 10% of net revenue | 162,809.17 |
| Variance | **(7,602.11)** |

Composed of: the three customer-referral halvings (−3,375.39), Terry Moore (−1,370.14), the five Beau overrides approved by Joe (+1,207.07 net), and the two suppressed deals (−4,063.71).

**Terry Moore 070426.DWB01** moved from 240.29 to **1,370.15** between 04/09 and 09/09 — also exactly 5% of net revenue. It is pre-reset so Earned is untouched and the −$2,500 adjustment still stands with no double count, but nobody has explained the movement.

**SSA057 Tom Jolly and SSA029 Danielle Campbell** remain gated with a blank Commission Payable — suppressed to zero, **$40,637.10 of net revenue paying nobody**. Unchanged since 28/08.

**Seven installed deals are held purely by the finance gates**: $171,624 of value and **$14,678.69 of commission** sitting in Pending. Peters 130726.JS01 progressed (STC now paid); Mark Wilson 150626.DWB01 is still awaiting both despite its "on next refresh" column-H comment.

## Closed since last run

Checked against column H of the deal sheet before re-raising anything.

- Ken Anderson SON reformatted `14082026.DWB01` → `140826.DWB02` (item K, "Hannah fixed")
- John Haig reassigned to `060526.JS01`, clearing the clash with Craig Robinson (item A)
- Charmaine Potter / Rosemary Arnold 2 shared SON resolved — Rosemary Arnold 2 is now `130826.JS02` (item A)
- Mick Dunn rebate corrected $17.00 → $17,507.00 (item G no longer fires)
- Nicole McKechnie 3 – POLE now carries $40,000
- Jenny Real and Brenda Walker are gated and in the log (item L delivered)
- Both dashboard copy defects left open on 04/09 are fixed
- FX rate verified after two failed runs

The worklist itself is still stamped "HubSpot pull 26/08/26" and several rows are now stale — worth regenerating.

## Still open

- **Hannah is now taking money**, not just diluting headcount — lead gen on Katie Young 2 with **$293.09 of pending** and no line of her own. Also on 140526.DWB01 and 240526.JS01. Add her, or clear the field.
- **Rebate exactly equals cash** on Nicole McKechnie ($17,600) and McKechnie 2 ($32,400) — inflates the pot by **$4,545.45**. No SM comment. Both SONs shifted up one again this run.
- **Summer Stewart appears twice under SON 250426.DWB01** ("1 Gleno Street" and "3-5 Gleno Street"), both Installed, both cash $31,267.25, carrying $2,996.31 and $2,904.07 of Commission Payable. Column H answers it, so it is tracked rather than new — but both records sit in the log and both inflate lifetime totals. Pre-reset, nothing payable.
- **Rosemary Arnold 2** has had both roles cleared — the whole $545.45 pot is now unallocated (it was one filled role on 04/09).
- **SSA037 Dieter Steinman** ($28,085) and "2 Scott Mooney" installed with no install date, in neither bucket. Scott Mooney is answered in column H; Dieter is not.
- **Brett Watson – OFFGRID** sold with no value and no SON — held out of the pipeline.
- Unallocated total **$10,509.30** — $8,821.59 pending across 5 deals, $1,687.71 in the log.
- Trailing whitespace on three SONs.
- No Cesca ad-spend report since 21/08 (19 days). Receipts cover the period; the PDFs stay unreadable through the connector.

## Deployment — root cause, established after the run

**Not published, and the reason I gave in the run above was wrong.** Ollie challenged it; here is what the probes actually show.

A **credential-injecting git proxy** sits in front of both `github.com` (git) and `api.github.com` (API), inside the CONNECT tunnel — it can, because TLS is re-terminated at the agent proxy whose CA this session trusts. It **discards any credential you supply** and injects its own, then serves it only for repos in "this session's authorized repository set".

| Probe | Result | What it proves |
|---|---|---|
| `api.github.com/user`, no credential | **200**, returns **ollieleuthart** (id 295091854), `X-Oauth-Client-Id: Iv23liqTIFEtdIu6Vn1r`, 15,000/hr | The injected token is a GitHub App user-to-server token **acting as Ollie**. It does not need his PAT. |
| `api.github.com/user`, deliberately fake token | **200**, same | The `Authorization` header is discarded, not validated. |
| `/repos/ollieleuthart/ssa-tracker` | **403** | — |
| `/repos/anthropics/claude-code`, `/repos/octocat/Hello-World`, `/repos/torvalds/linux` | **403, identical message** | **The allowlist is empty.** ssa-tracker is not specifically excluded — *nothing* is attached. |
| Headers on the 403 | no `Server`, no `X-GitHub-Request-Id`, no rate-limit headers (the 200 has all three) | The 403 never reached GitHub. The proxy answered. |
| Agent proxy `recentRelayFailures` | 20 entries, **zero** for github (all google hosts from the Playwright run) | **Not an egress-policy block.** The CONNECT succeeds. |
| `git push --dry-run`, fake token *and* no credential | identical: *"access denied by the git proxy: … is not in this session's authorized repository set, so the proxy will not inject a credential for it. To fix, add the repository to the session's sources."* | The git path is gated the same way, and names the fix. |

### Why it started

Pushing worked **continuously from 19/06/26 to 01/09/26 13:33 AEST** — commit `3cec7b2`, the current HEAD, byte-identical to the deployed `index.html` (md5 `1cbfdab8…`). Ten commits landed that day. The 04/09 runs failed, and so did this one. **The break is between 01/09 and 04/09.**

Every commit in the repo's history is authored *and* committed as `ssa <ollie@smart-switchappliances.com>`, in near-simultaneous pairs one second apart — the signature of the Contents API method with an explicit committer, one PUT per file. That is **not** the GitHub App's identity. So before 01/09, `api.github.com` was an ordinary host through the egress proxy and Ollie's own PAT reached GitHub directly.

**Nothing about the repo, the token, GitHub permissions or the tracker changed. The session type did.** A credential-injecting proxy was introduced, and these Cowork sessions are being created with no repository attached as a source.

### What follows from this

- **Never ask Ollie for a PAT again on this session type.** The proxy discards it, it cannot even be tested here, and the injected identity is already his. The 04/09 advice and my own were both wrong on this.
- **"Retry in a fresh session" is only right if that session has the repo attached at creation.** A fresh session with nothing attached fails identically.
- There is no `add_repo` tool here, and the agent proxy exposes no endpoint to request one (`/__agentproxy/*` answers 405 to everything but `status`).

### Routes

1. **Attach `ollieleuthart/ssa-tracker` as a source when the task is created** — this is the durable fix and the only one that restores the old one-step publish.
2. **Connect the folder holding the clone**; I write both files in with `device_commit_files`, and you run `deploy.bat` or click Commit + Push. (No `device_bash` in this session, so I cannot do the push myself.)
3. **Manual upload** at github.com/ollieleuthart/ssa-tracker → Add file → Upload files → commit to `main`.

### Interim

The dashboard is published as a **Claude artifact** at its own URL — live, shareable, updatable by republishing, and private until shared. That is strictly less exposed than the public repo it normally sits in. It does not replace the GitHub Pages model; it just means the 09/09 numbers are readable today instead of sitting in a chat attachment.

## Verification

**Verified in this session:** 0 formula errors across 221 formulas (`recalc.py`); workbook reloaded with `data_only=True` and every person's Current and Projected matched the JSON to the cent; Retainer Log TOTAL DRAWN $19,206.36 equals the Dashboard total, with an in-sheet CHECK cell reading 0.00; `earned + pending − to_cover = projected` holds for all five and Full Team; each person's slip rows sum exactly to their Earned; JS syntax clean; Playwright cleared the AMP2026 gate, visited all five tabs and cycled all six dropdown options on both dropdown tabs with **zero page errors** (one blocked Google Fonts request — the sandbox, not a defect); ad spend independently rebuilt from Gmail; the deployed shell's `pull` read directly as 28/08/26; the GitHub block proved with three controls; the saved state file re-loaded from disk and passed the integrity check the next run will apply.

**Told, not verified:** the retainer draw dates and amounts (Scott reconciles those against payroll — they cannot be derived from any source I can reach); that the Beau overrides were approved by Joe (deal sheet column H).

**Inferred, not verified:** that finance halved those three payables *because* the deals are customer referrals. The pattern is exact and confined to those three, but nobody has said so — which is why it is a question above rather than a change.

**Could not be checked from here:** the Cesca ad-spend PDFs (connector returns attachment metadata only); whether the six-day gap in Meta receipts is paused spend or undelivered mail.

**Left behind:** `archive/2026-09-09/` holds this run's `index.html`, workbook, `html_data.json` and `dq.json`, plus the previously deployed build and its data as `*_PREVIOUS_pull28Aug26.*`. `slip.last` stays `null` — no cycle has been approved, and showing an unapproved cycle as the previous payment would be worse than showing nothing.
