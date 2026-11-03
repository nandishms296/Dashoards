# Cost anomaly workbooks — Tableau 2023.3

Two packaged workbooks, same data and same calculated fields. They differ only in how the
band chart is built.

## Try this one first

**`metrics_max_anomaly.twbx`** — the band and the spend line on a dual axis, which is the layout
you were trying to build.

## If it won't open

**`metrics_max_anomaly_safe.twbx`** — no dual axis anywhere. The band is a Gantt chart on its own
sheet and the spend line is a separate sheet below it. Less elegant, far less likely to break.

Hand-written Tableau XML is version-sensitive and I had no Tableau to validate against, so treat
the first file as the optimistic attempt and the second as the fallback. If neither opens, the
calculated field formulas are all listed in `spike_dashboard_step_by_step.md`.

---

## What's inside

**Data** — 1,440 rows, two accounts, six services, July to 28 October 2026. Three deliberate
incidents are baked in so the detection has something to find:

| Account | Service | What it is |
|---|---|---|
| ITX-ACM | Amazon EFS | a **step change** from 11 Oct — spend triples and stays up |
| ITX-ACM | Amazon EC2 | a **two-day blip** around 17–18 Sep, z reaches 8.4 |
| ITX-AGS | Amazon Redshift | a shorter blip around 29 Sep |

Twenty-two account-days cross z = 2.0, which is roughly the alert volume you want.

**Calculated fields** — 13, already written and validated:

`Band Lower` · `Band Upper` · `Band Height` · `Account Daily Spend` · `Spike Flag` ·
`Spike Point` · `Latest Day Spend` · `Latest Day Expected` · `Latest Day Z` ·
`Latest Deviation` · `Latest Deviation %` · `Spike Days Last 14` · `Alert Level`

**Parameter** — `p_Z Threshold`, float, range 1.5 to 4.0, step 0.25, default 2.0.

**Sheets** — `C1 Daily band`, `K1 Spend`, `K2 Deviation`, `K3 Spike days`,
`C2 Service deviation`, `T1 Spike log`. Plus `C1b Spend line` in the safe variant.

**Dashboard** — `Cost anomaly monitor`, 1400 × 950.

---

## The band, and why it works here

The thing that kept failing was asking Tableau to give three measures different mark types on one
shared axis. It won't — Measure Values forces a single mark type.

The fix is a **Gantt bar**: a Gantt mark has a start and a length, which is exactly a floating bar
from the lower bound to the upper bound.

- `Band Lower` on Rows, mark type Gantt Bar
- `Band Height` on Size
- one mark card, no splitting, no stacking

If you ever rebuild this by hand, that's the whole trick.

---

## Switching to your Redshift view

1. Data → New Data Source → Redshift → `cost_usage_analytics.vw_spike_attribution`.
2. Data → the CSV source → **Replace Data Source** → pick the Redshift one.
3. Data → the CSV source → **Close**.

The CSV column names match the view exactly — `account_key`, `cost_date`, `billed`,
`baseline_avg`, `baseline_sd`, `delta_amt`, `delta_pct`, `z_score`, `service_flag`,
`account_billed`, `account_baseline`, `account_baseline_sd`, `account_delta`,
`account_delta_pct`, `account_z`, `contribution_share`, `as_of_date` — so every calc, filter and
sheet survives the swap.

The one column the CSV has that the view doesn't is nothing; the view has extras (`delta_rank`,
`services_active`, `product_family`, `project`) which simply become available afterwards.

---

## What still needs doing by hand

I kept the XML deliberately plain, so the styling isn't there:

- Colours — Spike `#D62728`, Normal `#1F4EDB`, band grey `#D9D9D9` at 25% opacity
- Number formats — currency 0dp on the spend measures, 2dp on `account_z`
- The 90-day date filter on `C1 Daily band`
- Tooltips
- The attribution waterfall, price/volume split and heatmap — all covered step by step in
  `spike_dashboard_step_by_step.md`

None of that affects whether the file opens, which is why it's left out.
