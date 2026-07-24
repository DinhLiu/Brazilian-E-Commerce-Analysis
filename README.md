# Olist Analytics

End-to-end analysis of the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — cleaning, exploratory analysis, customer segmentation, delivery/CX diagnostics, and a Streamlit dashboard.

**Scope:** ~99k orders, ~95k customers, ~3.1k sellers (Sep 2016 – Aug 2018).

---

## Key Findings

Full write-up: [`reports/insights_summary.md`](reports/insights_summary.md)

1. **Single-intent marketplace** — ~97% of customers buy once; ~97% of orders are single-product. Cart bundling has little leverage.
2. **Value concentration** — “Cannot Lose Them” is 14.7% of customers but **27.5% of revenue** (top CRM priority).
3. **Promise > speed** — Late deliveries cut review scores by ~2 points; among on-time orders, absolute speed only moves scores by ~0.5.
4. **Northeast CX is partly ETA calibration** — Thin ETA buffers align with the highest late rates (AL 21%, MA 18%, SE 15%). ~78% of late orders are carrier-side, not seller handoff failures.

---

## Project Structure

```
olist-analytics/
├── data/
│   ├── raw/                         # Original Olist CSVs
│   └── processed/                   # Cleaned parquet outputs from notebooks
├── notebooks/                       # Analysis pipeline (run in order)
│   ├── 01_data_cleaning.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_rfm_segmentation.ipynb
│   ├── 04_delivery_review_analysis.ipynb
│   ├── 05_market_basket.ipynb
│   ├── 06_seller_category_performance.ipynb
│   ├── 07_repeat_purchase_drivers.ipynb
│   ├── 08_eta_calibration_by_state_seller.ipynb
│   └── 09_seller_fulfillment_SLA.ipynb
├── src/                             # Reusable helpers used by notebooks
│   ├── data_loader.py               # Load raw CSVs & join item/order tables
│   ├── data_cleaning.py             # Timestamp checks, missing/outlier handling
│   └── features.py                  # Delivery features, RFM scoring
├── dashboard/
│   ├── app.py                       # Streamlit app
│   └── loaders.py                   # Cached loaders for processed parquet
├── documents/                       # ERD & schema diagrams
├── reports/
│   └── insights_summary.md          # Consolidated business insights
├── pyproject.toml
├── requirements.txt
└── LICENSE
```

---

## Setup

Requires **Python ≥ 3.10**.

```bash
# Clone and enter the repo
cd olist-analytics

# Create and activate a virtualenv
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# Install dependencies (+ editable package so `import src` works)
pip install -r requirements.txt
pip install -e .
```

### Data

Place the Olist CSVs under `data/raw/`:

| File |
|------|
| `olist_customers_dataset.csv` |
| `olist_geolocation_dataset.csv` |
| `olist_order_items_dataset.csv` |
| `olist_order_payments_dataset.csv` |
| `olist_order_reviews_dataset.csv` |
| `olist_orders_dataset.csv` |
| `olist_products_dataset.csv` |
| `olist_sellers_dataset.csv` |
| `product_category_name_translation.csv` |

Dataset: [Kaggle — Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Schema / ERD: see [`documents/`](documents/).

---

## Run the Analysis

Notebooks are numbered — run them **in order**. Each step writes parquet (and related) outputs under `data/processed/` for the next notebooks and the dashboard.

| # | Notebook | What it does |
|---|----------|--------------|
| 01 | Data cleaning | Load raw tables, clean, join → `item_level` / `order_level` |
| 02 | EDA | Delivery features, monthly volume, Sep 2018 truncation |
| 03 | RFM segmentation | Recency / frequency / monetary segments & revenue share |
| 04 | Delivery vs review | Lateness, delay buckets, state-level CX |
| 05 | Market basket | Co-purchase rules on multi-item orders |
| 06 | Seller & category | Revenue/growth by category; high-revenue / low-review sellers |
| 07 | Repeat purchase drivers | First-order factors vs category repeat rates |
| 08 | ETA calibration | Promised vs actual delivery by state & seller |
| 09 | Seller fulfillment SLA | Seller handoff vs carrier delay attribution |

Launch Jupyter from the project root (so `src` imports resolve):

```bash
jupyter notebook notebooks/
# or
jupyter lab notebooks/
```

---

## Dashboard

Interactive Streamlit app covering overview, RFM, repeat drivers, delivery/reviews, ETA calibration, categories & sellers, fulfillment SLA, and market basket.

```bash
# Requires data/processed/ outputs from the notebooks
streamlit run dashboard/app.py
```
## Live Dashboard
[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://<tên-app>.streamlit.app)


---

## Processed Outputs

Main artifacts written under `data/processed/`:

| File | Source |
|------|--------|
| `item_level.parquet` / `order_level.parquet` | 01 |
| `order_level_trend.parquet` | 02 |
| `rfm.parquet` | 03 |
| `market_basket_rules.parquet` | 05 |
| `category_summary.parquet` / `category_growth.parquet` | 06 |
| `seller_summary.parquet` / `risk_sellers.parquet` | 06 |

---

## Stack

- **Data:** pandas, pyarrow / fastparquet, numpy
- **Stats / ML:** scipy, statsmodels, mlxtend
- **Viz:** matplotlib, seaborn, plotly
- **App:** streamlit

---

## License

See [`LICENSE`](LICENSE). The Olist dataset is published separately under its own terms on Kaggle.
