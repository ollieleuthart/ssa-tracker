# SSA Commission Tracker — Formatting & Construction Guide

**Version 1.0 · 28/07/26**
Reference document for building and maintaining the SSA commission tracker (HTML dashboard + xlsx workbook).
Upload to project files so every future build follows the same rules.

---

## 1. What this system is

Two linked deliverables, always built from the same computed dataset so their numbers can never disagree:

| Deliverable | File | Purpose | Audience |
|---|---|---|---|
| HTML dashboard | `index.html` (live) / `reset_dashboard.html` (working) | Live self-serve view | Sales team |
| Workbook | `SSA_Tracker_Reset_20Jul2026.xlsx` | Sendable, auditable | Team + finance |

**Golden rule:** compute figures **once** into a single JSON structure, then render both outputs from it. Never calculate separately per format.

---

## 2. Data source — HubSpot

HubSpot is the **sole source of truth** for deal data. The Legacy Commission Log is retired (double-counting risk).

### 2.1 Deal properties used

| Property | Meaning | Notes |
|---|---|---|
| `dealname` | Deal name | Usually "Customer - Address" |
| `son` | **Sales Order Number** | For Xero cross-reference. **Always pull this.** |
| `installation_status` | Installed / Upcoming / In Progress / Cancelled / On Hold | Drives earned vs pending |
| `date_installed` | Install date | Often blank even when Installed — see §8 |
| `amount` | Cash (incl GST) | |
| `rebate_amount` | Rebate (incl GST) | |
| `lead_gen` | Lead gen people (`;` separated) | **Value corruption — trust labels** |
| `specialist` | Specialists (`;` separated) | Clean, no corruption |
| `deal_currency_code` | Always AUD | |

### 2.2 Known HubSpot traps

- **`properties` must be listed explicitly.** Omitted fields return silently empty — they do **not** error. This caused a real mistake: probing for `sales_order_number` returned nothing and led to the wrong conclusion that no SO field existed. **Always use `search_properties` with keywords to discover real field names before concluding a field is absent.**
- **`lead_gen` dropdown value corruption:** internal value `Simon` → label `Fran`; `No comms` → label `Dermot`. Trust the **label**, not the value. `specialist` is unaffected.
- **`xero_amount_*` GL fields exist in the schema but are entirely empty.** Ignore.
- Other real-but-sparse fields: `order_no`, `customer_ref_no`, `stock_ordered`.

### 2.3 Standard pull

```
search_crm_objects(
  objectType="deals",
  filterGroups=[{filters:[{propertyName:"installation_status",
                           operator:"IN",
                           values:["Installed","Upcoming","In Progress"]}]}],
  properties=["dealname","son","installation_status","date_installed",
              "amount","rebate_amount","deal_currency_code",
              "lead_gen","specialist","dealstage"],
  limit=200,
  sorts=[{propertyName:"date_installed",direction:"DESCENDING"}]
)
```
Cancelled / On Hold are **excluded** entirely.

---

## 3. Commission rules

### 3.1 Core formula

```
Base   = (Cash + Rebate) ÷ 1.1        # strips GST
Total  = Cash + Rebate                # incl GST
```

**Away trip** — 3 or more distinct field reps on the deal (Holly excluded from the headcount):
```
Per person = (Base × 10%) ÷ headcount
```

**Home deal** — fewer than 3 field reps. Roles are separate and additive:
```
Lead gen share    = (Base × 5%) ÷ number of lead gen people
Specialist share  = (Base × 5%) ÷ number of specialists
```
A person in both roles earns **both** shares. This is a 5% + 5% role split — **not** a single 10% headcount pool.

### 3.2 Holly (Marketing / online lead gen) — special case

```
Holly = Cash ÷ 1.1 × 0.8 × 5%
```
- ÷1.1 removes GST, ×0.8 removes 20% finance charges, ×5% is her LG rate
- **Cash only — rebate excluded**
- She appears in HubSpot as `Social Media`
- Excluded from away-trip headcount
- Retainer $250/week; Meta ad spend logged as retainer rows under her name

### 3.3 Name normalisation

| HubSpot value | Maps to |
|---|---|
| `Max B` | Max Bradfield |
| `Cian` | Cian Jefferson |
| `Denzel` | Denzel Winterbeard |
| `Mark` / `Poochy` | Mark Isaac *(always merge)* |
| `Social Media` | Holly |
| `No comms` / `Customer` | *(blank — no commission)* |
| Hugo, Rosa, Fran, Harley, Dermot, Joe, Beau Jolly, Simon, Hannah | unchanged |

Deduplicate after mapping (same person can appear twice via different aliases).

### 3.4 Currency — Meta ad spend

NZD → AUD at the rate in Retainer Log Settings (**currently 0.8284**). **AUD amounts stay AUD — do not convert.** Reports split by ad account: Joe Scott account = NZD, SSA-AU account = AUD.

---

## 4. The 20 July 2026 reset model

Inception date **20/07/26**. Only five people. Commission counts **forward only**; all prior history is baked into each person's starting position.

| Person | Type | Start | Notes |
|---|---|---|---|
| Denzel Winterbeard | FTE (Salaried) | –$4,789.06 | From 20/07 commission slip |
| Hugo | FTE (Salaried) | –$7,730.41 | From slip; reclassified from contractor |
| Beau Jolly | Independent Contractor | $0 | Commission-only |
| Joe | FTE (Salaried) | $0 | See §11 — internal handling |
| Holly | Independent Contractor (Marketing) | $0 | Retainer/advance applies |

### 4.1 Position formulas

```
Current    = Start + Earned − Retainer
Projected  = Current + Pending
           = Start + Earned + Pending − Retainer
```

- **Earned** = deals with `installation_status = Installed` AND `date_installed >= 2026-07-20`
- **Pending** = deals with status `Upcoming` or `In Progress` (any date)
- Deals installed **before** 20/07/26 are excluded from the reset tracker entirely (they live in the Historic Deals Log only)

### 4.2 FTE display rule

FTE deficits are shown as **debt to earn back** (raw slip figure, not floored at zero). Ongoing retainer of **$1,250/wk gross** ($65k/yr) **adds to the deficit** — log weekly draws in the Retainer Log so the line moves.

Reconciliation: $1,250 gross/wk = $1,026 net + $224 PAYG + $150 super → $2,052 net/fortnight, matching the slips. Super (12%) is payable **in addition**, via payroll, not calculated in the tracker.

### 4.3 Status labels

```
Positive → "✅ SSA OWES"
Negative → "⛔ TO EARN BACK"
```

---

## 5. Design system — HTML dashboard

### 5.1 Colour tokens (exact)

```css
--bg:#0F1A2E;        --surface:#172137;   --surface2:#1E2B45;  --surface3:#243350;
--border:#2A3A58;    --text:#E2E8F0;      --text2:#94A3B8;     --text3:#64748B;
--teal:#3ECDC6;      --teal-dim:rgba(62,205,198,.15);
--green:#34D399;     --blue:#60A5FA;
--amber:#FBBF24;     --amber-dim:rgba(251,191,36,.12);
--red:#F87171;       --purple:#A78BFA;
```

### 5.2 Semantic colour rules — one meaning per colour

| Colour | Means | Used for |
|---|---|---|
| **Red** `--red` | Owed to SSA / to earn back | Negative positions, deficit line, "TO EARN BACK", negative bars |
| **Green** `--green` | Money owed to the person | Earned amounts, positive positions, positive bars |
| **Blue** `--blue` | Pending / not yet installed | Pending column header **and** Projected header, pending values, pending bar segment, pending KPI, column divider |
| **Teal** `--teal` | Brand / identity | Logo, SON values, section accents, Historic "Commission" header |
| **Amber** `--amber` | Neutral warning — a fact, not a debt | Retainer/advance KPI, data-quality flags, flagged row tint |

**Never** use amber for owed amounts, and never use it as a column-header theme. Values always carry their own sign colour independent of the header.

### 5.3 Typography

- **Inter** (Google Fonts), weights 400/500/600/700/800
- Monospace (`ui-monospace, Menlo, monospace`) for **SON values** and eyebrow labels
- Base 14px; hero value 38px/800; KPI value 21px/700; table 12.5px

### 5.4 Component spec

**Password gate** — full-screen overlay, SHA-256 hashed. Password `SSA2026`, hash:
```
a41594e694e3b6814ea409311f61cf9cb2fca4708bb60bd67acb3f2bd00a8979
```
⚠ Client-side only. Data is embedded in page source and readable without the password. See §12.

**Header** — logo left, date badge right:
```
RESET · 20/07/26 baseline · pulled DD/MM/YY
```

**Tab nav** — pill-style, teal active state, `switchView(id, event)`:
1. Pipeline Forecast
2. My Position
3. Team List
4. Historic Deals Log

**Hero panel** — left accent bar (teal positive / red negative), eyebrow · value · caption.

**KPI cards** — 4 across, 3px top accent bar, label / value / sub-label.

**Progress bar** — green earned, blue pending, grey remaining, red vertical line at the target with `data-label` showing the amount. Dollar tick scale beneath (`niceStep()` → 1/2/2.5/5/10 × power of 10, plus an end tick if the last step is under 93%).

### 5.5 Copy conventions

**One factual template for every person — no per-person storytelling.** Hero caption:

```
{Name} has {pending} in pending commission still to install.
Once it does, {SSA will owe {Name} | {Name} will owe SSA} {amount}.
```

Full Team variant swaps in "The team" / "the team" and appends "in total".

Rules:
- State facts, not narrative. No reassurance, no editorialising.
- Identical sentence structure for everyone; only numbers and direction change.
- Never add explanatory commentary about employment status or entitlements.

### 5.6 Dates

**`dd/mm/yy` everywhere.** Use the `fmtDate()` helper for ISO input:
```js
const fmtDate=s=>{if(!s||s==="—")return "—";
  const m=String(s).match(/^(\d{4})-(\d{2})-(\d{2})$/);
  return m?`${m[3]}/${m[2]}/${m[1].slice(2)}`:s;};
```
Month-only references use `mm/yy` (e.g. "since 09/25"). Never long-form month names.

### 5.7 Number formatting

```js
fmt(v)   // no decimals:  $1,234  /  ($1,234)
fmt2(v)  // 2 decimals:   $1,234.56  /  ($1,234.56)
```
Negatives in parentheses, never a minus sign. Tabular-nums for alignment.

---

## 6. Dashboard tab specification

### Tab 1 — Pipeline Forecast
- 4 KPIs: Pipeline value · Pending commission · Earned since 20/07/26 · Deals in pipeline
- Horizontal comparison bars, one per person (**Full Team excluded** — it would dwarf the scale)
- Deals table: **SON** · Deal Name · Lead Gen/Specialist · Type · Stage · Deal Value · Comm/Person

### Tab 2 — My Position
- Dropdown: **Full Team first (default)**, then individuals
- Badge: "Whole team · aggregate" / "Employee · draw" / "Contractor"
- Hero · 4 KPIs · progress bar with scale
- Deals table: **SON** · Deal · Stage · Type · Their share
- Full Team shows all deals with combined commission per deal

### Tab 3 — Team List
- Columns: Name · Type · Start · Retainer · Earned · Current · ⟨blue divider⟩ Pending · Projected
- Full Team row **excluded** (totals would duplicate)
- Footnote defining `Projected = Current + Pending` plus the colour key

### Tab 4 — Historic Deals Log
- All installed deals back to 09/25
- 4 summary boxes: Total installed value · Total commission · Deals logged · Flagged rows
- **Amber estimate caveat** immediately above the table (see §8)
- Columns: **SON** · Deal · Installed · Lead Gen/Specialist · Type · Deal Value · Commission · Flag
- Flagged rows tinted `rgba(251,191,36,.07)`
- Sorted newest first, undated last

---

## 7. Workbook (xlsx) specification

### 7.1 Tab order

1. `📊 Dashboard`
2. `🔮 Pipeline`
3. `💰 Retainer Log`
4. `📜 Historic Deals Log`
5. `📋 READ ME`

### 7.2 Conventions

- **Header rows:** navy `#0F1A2E` fill, white bold 10pt Arial, centred, wrapped
- **Input cells:** blue font `#0000FF` (signals user-editable)
- **Money format:** `$#,##0.00;($#,##0.00);-`
- **SON column:** Consolas 10pt, teal `#0F7A75`, always first column
- **Flagged rows:** amber fill `#FEF3C7`
- **Aggregate rows:** grey fill `#F1F5F9`, bold
- **Freeze panes** below the header row on every data tab
- Every tab carries the pull date in its subtitle

### 7.3 Dashboard columns

```
A Name | B Type | C Start Position | D Retainer Paid | E Commission Earned
F Current Position | G Status | H Commission Pending | I Projected Position | J Status Projected
```
Formulas: `F = C+E−D`, `I = C+E+H−D`, statuses `=IF(F5>=0,"✅ SSA OWES","⛔ TO EARN BACK")`.
**FULL TEAM** row at the bottom uses `=SUM()` across all people — never a hardcoded cell list (this has broken before when a person was added).

### 7.4 Validation — mandatory before delivery

```bash
python3 /mnt/skills/public/xlsx/scripts/recalc.py <file>.xlsx
```
Must report **0 errors**. Then reload with `data_only=True` and verify figures actually resolve — a formula can be error-free and still point at the wrong cell.

---

## 8. Data quality — known issues

Currently **12 flagged rows** of 74 installed deals. Flag types:

| Flag | Meaning |
|---|---|
| `no install date` | Status Installed but `date_installed` empty |
| `no value` | Cash and rebate both zero/blank |
| `no attribution` | No lead gen and no specialist recorded |
| `date outside range` | Install date before Sept 2025 (e.g. Larry Grant, Feb 2025) |

**Never silently patch these.** Surface them: amber row tint + named flag in a Flag column, and a count in the summary.

**Mandatory caveat wording** on any pre-reset totals:

> ⚠ **Estimate up to 20/07/26.** Figures before the reset are reconstructed from HubSpot and contain known inconsistencies — some deals have no install date, no deal value, or no attribution recorded (N rows flagged). Commission shown is **earned/owed on install**, not cash paid. Treat pre-reset totals as an estimate for cross-referencing, not a settled ledger.

Also note: **SONs are not unique.** Multi-property orders share one SON (e.g. both Mansfield properties). Don't use SON as a primary key.

---

## 9. Build & refresh process

1. **Pull HubSpot** — full active set with the property list in §2.3
2. **Normalise names**, map aliases, dedupe
3. **Compute commissions** via the shared engine (§3) — Holly override applied after the generic calculation
4. **Split** into earned (installed ≥ 20/07/26) vs pending (Upcoming / In Progress)
5. **Apply** starting positions and retainer draws
6. **Build Full Team aggregate** = sum of the five
7. **Write `html_data.json`** — the single source both outputs render from
8. **Render HTML**, then **render xlsx**
9. **Validate:**
   - `node --check` the extracted `<script>` block
   - Playwright render — check every tab, every dropdown option, assert zero console/page errors
   - `recalc.py` on the xlsx — 0 errors
   - Reload xlsx `data_only=True` and eyeball resolved values
10. **Stamp the pull date** (`DD/MM/YY`) in the HTML badge and all xlsx subtitles — **every time**
11. **Deliver** both files, then deploy if asked

### 9.1 Shared engine

Keep `engine.py` as the single implementation of `norm_list()`, `deal_commissions()`, and the Holly override. Both renderers import it. Never duplicate the logic.

---

## 10. Deployment

**GitHub Pages** — repo `ollieleuthart/ssa-tracker`, branch `main`, file `index.html`.
Live: `https://ollieleuthart.github.io/ssa-tracker/`

Process: `GET /repos/{owner}/{repo}/contents/index.html` → take `sha` → `PUT` with base64 content + that sha + commit message. Then poll `/pages/builds/latest` until `status: built`. Verify the deployed bytes match local (MD5).

Use a fresh personal access token each time; **revoke after use**. Never leave a token live in chat history.

**Google Drive** — Commission Tracker folder: `11VGepKVBw34zqroODRUOrSL45fUBcPlt`
Subfolder "Updated from 20th July": `1hTxjMmsd6Og4IL6wkN-CIiHCcHV38ZZE`
Connector is **ID-based only** — no folder listing, no search. Always ask for a direct file/folder link. Upload xlsx with `disableConversionToGoogleType: true` to keep it native.

---

## 11. Internal handling — not for distribution

> **Keep this section out of any file shared with the team.**

**Joe** is displayed as a normal FTE throughout both deliverables and is included in all totals, but he is **not paid commission**. His figures are notional/tracking only. Every build must render him identically to any other team member — no badge, no note, no exclusion from totals, nothing in the page source. Finance must be told separately to net him out; the files will not indicate it.

---

## 12. Security note

The password gate is **cosmetic**. It hides a div; it does not protect the data. The full dataset is embedded as plain-text JSON in the page source, and the password hash (unsalted SHA-256 of a short string) is embedded too. On a public repo, anyone with the URL can read every figure via View Source.

Accepted for now. If this needs to change: Netlify or Cloudflare Pages free tier offers real server-side password protection; alternatively strip individual positions from the published version and keep them in the xlsx sent directly.

---

## 13. Resolved decisions — do not revisit

- Poochy = Mark Isaac (always merge)
- Simon standardised to "Simon" (not "Simon White")
- Holly: ÷1.1 GST **and** ×0.8 finance charges, **cash only**, rebate excluded
- HubSpot is sole source of truth; Legacy Commission Log retired
- Away threshold = 3+ field reps
- Commission is 5% + 5% role split, not a 10% headcount pool
- FTEs shown a debt to earn back (Fair Work concern raised and **cleared by management** — do not re-flag)
- Hugo reclassified contractor → FTE
- Reset date 20/07/26; only five people
- Full Team is the default dropdown option
- Dates `dd/mm/yy` throughout
- Pull date refreshed on every rebuild
- GitHub Pages public + password gate = accepted deployment model
- Projected header uses the same blue as Pending

---

## 14. Quick checklist

```
[ ] HubSpot pulled fresh, `son` included in properties
[ ] Names normalised and deduped
[ ] Holly override applied after generic calc
[ ] Earned = installed ≥ 20/07/26 only
[ ] Full Team aggregate rebuilt
[ ] SON column present on EVERY deal table
[ ] All dates dd/mm/yy
[ ] Pull date stamped in HTML badge + xlsx subtitles
[ ] Hero copy uses the standard template
[ ] Colour semantics: red=owed, green=owed to them, blue=pending, amber=retainer/flags
[ ] Data-quality flags surfaced, caveat present on pre-reset totals
[ ] Joe rendered as a normal team member (§11)
[ ] node --check passes
[ ] Playwright: all tabs, all dropdown options, zero errors
[ ] recalc.py: 0 errors, values verified with data_only=True
[ ] Both files delivered
```
