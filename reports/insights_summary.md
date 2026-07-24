# Olist Marketplace — Insights Summary

Analysis based on notebooks `01`–`09` (cleaned Brazilian e-commerce orders through Aug 2018). Scope: ~99k orders, ~95k customers, ~3.1k sellers.

---

## Executive Summary

Olist is a **single-intent marketplace**: ~97% of customers buy once, and ~97% of orders contain a single product. Growth and retention cannot rely on Amazon-style basket building.

Three findings dominate the business case:

1. **Value is concentrated in a quiet high-spender segment.** “Cannot Lose Them” is 14.7% of customers but **27.5% of revenue** — the #1 CRM priority.
2. **Broken delivery promises hurt more than slow shipping.** Late orders score ~2 points lower than on-time; among on-time orders, absolute speed only moves reviews by ~0.5 points.
3. **Northeast CX pain is partly controllable via ETA calibration.** Thin ETA buffers align with the highest late rates (AL 21%, MA 18%, SE 15%). Platform-wide, **~78% of late deliveries are carrier-side**, not seller handoff failures.

---

## 1. Dataset & Scope

| Topic | Finding |
|-------|---------|
| Data quality | Generally good; 1,382 orders with timestamp violations (mostly approved → carrier). Flagged, not dropped. |
| Time series | Stable ~6–7k orders/month in early–mid 2018. Sep 2018 has 1 order (collection cutoff) — **exclude from trends**; keep through Aug 2018. |
| Customer IDs | `customer_unique_id` uniqueness ≈ 96.6% — true customers fewer than order-level IDs. |
| Reviews | Scores valid 1–5; ~41% include a written comment. |

---

## 2. Customer Retention & RFM

**94,983** customers (after removing canceled/unavailable). Mean recency ~243 days; mean frequency **1.03**; mean monetary ~166 BRL. **96.96%** of customers purchase only once — frequency is nearly useless for scoring; monetary is right-skewed (use log).

| Segment | Customers | % Cust | % Revenue | Avg monetary (BRL) |
|---------|-----------|--------|-----------|--------------------|
| New Customers | 36,752 | 38.7% | 38.5% | 164.7 |
| Hibernating | 22,970 | 24.2% | 10.6% | 72.8 |
| Need Attention | 18,386 | 19.4% | 17.8% | 152.1 |
| **Cannot Lose Them** | **13,988** | **14.7%** | **27.5%** | **309.1** |
| Loyal | 1,723 | 1.8% | 3.3% | 298.5 |
| At Risk | 1,035 | 1.1% | 1.9% | 295.5 |
| Champions | 129 | 0.14% | 0.45% | 543.6 |

**Insights**
- **Cannot Lose Them** — high spenders gone quiet; highest leverage for win-back (personalized high-value offers).
- **New Customers** — largest volume, low AOV; second-purchase incentives can lift LTV.
- **Champions** — tiny but ~3.3× New Customer AOV; VIP / referral program.
- **Hibernating** — large and low-value; low-cost automated drip only.

### What actually drives repeat?

Platform repeat rate is only **3.03%**. First-order logistics gaps between one-time and repeat buyers are small (late rate 6.7% vs 5.6%; review 4.10 vs 4.15; delivery days nearly identical). **Category matters more than first-order service quality.**

**Reliably higher repeat** (CI above ~3% baseline):

| Category | Repeat rate | Notes |
|----------|-------------|-------|
| home_appliances | 8.9% | CI 7.0–11.2% |
| fashion_bags_accessories | 6.2% | n=1,886 |
| furniture_decor | 5.0% | n=7,918 |
| bed_bath_table | 4.9% | n=10,523 |

**Reliably lower:** electronics (1.6%), musical_instruments (1.5%). Drop “0% repeat” categories with n≈75–99 — sampling noise.

**Implication:** Target win-back at customers whose first purchase was in home/fashion categories; do not assume service improvements alone will create repeat.

---

## 3. Delivery, Reviews & Geography

Analysis set: **95,103** delivered orders with valid timestamps. Overall late rate **6.8%** (6,535 / 96,476 delivered).

| Condition | Avg review |
|-----------|------------|
| On-time | **4.29** |
| Late | **2.27** (Δ −2.02; Mann–Whitney p ≈ 0) |

| Delay bucket | Avg review | n |
|--------------|------------|---|
| Soon / on time | 4.29 | 88,106 |
| Late 1–3 days | 3.29 | 1,842 |
| Late 4–7 days | 2.10 | 1,741 |
| Late 8–14 days | **1.67** | 1,445 |
| Late >14 days | **1.72** | 1,330 |

Among **on-time** orders only, absolute speed: 4.43 (≤5 days) → 3.92 (>20 days) — ~0.5 pt drop. **Expectation violation hurts far more than slow-but-honest delivery.** Pain saturates around **8 days late**.

**Geography:** state late_rate ↔ avg_review correlation **−0.876**. Worst late rates: AL **21.1%**, MA **17.7%**, SE **15.3%** (Northeast cluster with PI, CE). SP late rate **4.5%**.

---

## 4. ETA Calibration & Fulfillment Attribution

### ETA bias
Median ETA error **−12 days** (conservative buffer) — explains the low platform late rate. States with the **least buffer** match the worst late rates: AL (−8.4d / 21%), MA (−8.9d / 18%), SE (−9.4d / 15%). SP has a similar buffer (−10.4d) but much lower lateness — infrastructure absorbs the same nominal buffer better.

| Correlation (by state) | Value |
|------------------------|-------|
| eta_error ↔ late_rate | **0.675** |
| eta_error ↔ avg_review | **−0.426** |
| late_rate ↔ avg_review | **−0.876** |

**14 of 18** previously flagged high-revenue/low-review sellers also appear in the tight-ETA list — independent confirmation.

**Action:** Widen ETAs for Northeast (AL, MA, SE, CE, BA, PI) and thin-buffer sellers — low-cost vs building logistics.

### Who causes delay?
| Attribution | Share of late orders |
|-------------|----------------------|
| Carrier | **78.4%** |
| Seller | 12.2% |
| Both | 9.4% |

Seller SLA miss rate overall: **4.72%** (median seller_delay **−4 days** — sellers usually hand off early). Seller miss rates are ~5–6% across states — Northeast is **not** a seller-prep hotspot.

**Seller triage (heterogeneous risk list)**
- True fulfillment problems: e.g. `54965bbe…` (43.7% miss, +8.5d), plus several at 30–40% miss → SLA enforcement.
- Largest revenue risk seller `7c67e1448b…` (#5 revenue, 186k BRL, review 3.35): only **14.4%** miss and ships early (−2.5d) → **not** a speed problem; probe product quality / assigned carrier.
- Exclude 4 low-n risk sellers from SLA conclusions.

**Platform priority:** carriers + honest ETAs, not blanket seller enforcement.

---

## 5. Marketplace Structure (Categories, Sellers, Basket)

### Categories
Top revenue (cash cows, ~flat growth): health_beauty (~1.21M BRL), watches_gifts (~1.15M), bed_bath_table (~1.01M), sports_leisure, computers_accessories.

High growth sits in small-revenue niches (e.g. arts_and_craftmanship, construction_tools_lights, diapers_and_hygiene). Treat extreme growth ratios carefully (small-n artifacts).

### Seller quality risk
**18** high-revenue sellers with avg review **&lt; 3.5**. Audit before scaling; start with `7c67e1448b…`, then split by SLA-miss vs other root causes (see §4).

### Market basket
- **96.72%** single-product orders; only 3,236 multi-item (~3.28%).
- Only meaningful rule: `home_confort` → `bed_bath_table` (lift **3.0**, confidence **74.1%**, n=43); reverse confidence **5.4%**.

**Strategy:** Prefer **post-purchase / email** cross-sell over in-cart bundling. A/B-test the one validated pair; do **not** build a large-scale bundle system.

---

## 6. Prioritized Recommendations

| Priority | Action | Source |
|----------|--------|--------|
| **P0** | Win-back **Cannot Lose Them** (personalized high-value offers) | 03 |
| **P0** | Widen / recalibrate ETAs for Northeast + thin-buffer sellers | 04, 08 |
| **P1** | Second-purchase program for New Customers; prioritize home/fashion first categories | 03, 07 |
| **P1** | Proactive CX when delay crosses **7–8 days** (notify / compensate) | 04 |
| **P1** | Triage risk sellers: SLA-miss subset vs `7c67…`-type quality/carrier issues | 06, 08, 09 |
| **P2** | Regional logistics / local carriers for Northeast (AL, MA, SE, PI, CE) | 04, 09 |
| **P2** | Light A/B: home_confort → bed_bath_table cross-sell | 05 |
| **P3** | Champion VIP / referral; low-cost Hibernating drip | 03 |
| **P3** | Promote early-stage growth categories; maintain (don’t over-invest) cash cows | 06 |

---

## Appendix — Notebook Map

| Notebook | Role |
|----------|------|
| 01 Data cleaning | Load, clean, join → `data/processed/` |
| 02 EDA | Delivery features, volume trend, Sep 2018 truncation |
| 03 RFM segmentation | Segments, revenue concentration, CRM priorities |
| 04 Delivery vs review | Lateness, delay buckets, state geography |
| 05 Market basket | Single-item behavior, one category association |
| 06 Seller & category | Cash cows vs growth niches, risk sellers |
| 07 Repeat drivers | First-order factors vs category repeat rates |
| 08 ETA calibration | Buffer bias by state/seller; link to late rate & reviews |
| 09 Seller fulfillment SLA | Seller vs carrier delay attribution; risk-seller triage |
