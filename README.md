# Cross-Channel Marketing Performance Analysis

> A unified data model and analytics pipeline for comparing paid ad performance across Facebook, Google Ads, and TikTok — built as part of the Improvado Marketing Analyst Technical Assignment.

---

## Project Overview

Each ad platform exports data in its own schema. This project normalizes all three into a single unified table using BigQuery SQL, enabling apples-to-apples cross-platform KPI comparisons.

**Data period:** January 2024 &nbsp;|&nbsp; **Total records:** 330 rows (110 per platform)

---

## Repository Structure

```
├── 01_facebook_ads.csv             # Raw Facebook Ads export — Jan 2024
├── 02_google_ads.csv               # Raw Google Ads export — Jan 2024
├── 03_tiktok_ads.csv               # Raw TikTok Ads export — Jan 2024
├── unified_model.sql               # Core BigQuery SQL — normalizes & unions all platforms
├── unified_data.csv                # Output of unified_model.sql — 330 rows, ready for BI
├── analysis.py                     # Local Python validation — mirrors all BigQuery views
├── dashboard.html                  # Self-contained HTML dashboard (no server needed)
├── LOOKER_STUDIO_SETUP.md          # Step-by-step Looker Studio connection guide
└── views/
    ├── 00_qa_validation.sql        # Data quality checks
    ├── 01_platform_summary.sql     # Per-platform KPI summary cards
    ├── 02_campaign_performance.sql # Campaign leaderboard + scatter
    ├── 03_daily_trend.sql          # Daily spend & CPA trends
    ├── 04_tiktok_funnel.sql        # TikTok video completion funnel
    ├── 05_quality_score_analysis.sql # Google Quality Score vs CTR
    ├── 06_budget_vs_conversion.sql   # Budget share vs conversion share
    ├── 07_tiktok_video_funnel_widget.sql
    ├── 08_google_quality_score_widget.sql
    ├── 09_weekly_cpa_trend_widget.sql
    └── 10_efficiency_scorecard_widget.sql
```

---

## The Problem

Each platform uses different column names for the same concepts:

| Concept | Facebook | Google Ads | TikTok |
|---|---|---|---|
| Money spent | `spend` | `cost` | `cost` |
| Ad group ID | `ad_set_id` | `ad_group_id` | `adgroup_id` |
| Video views | `video_views` | — | `video_watch_25/50/75/100` |
| Platform-specific | `reach`, `frequency` | `quality_score`, `search_impression_share` | `likes`, `shares`, `comments` |

Without normalization, cross-platform comparison is impossible.

---

## Solution: Unified Data Model

`unified_model.sql` is a BigQuery Standard SQL query that:

1. **Renames** platform-specific columns to a shared schema
2. **Fills NULLs** for columns that don't apply to a given platform
3. **Calculates KPIs** at the SQL layer for consistency:
   - **CTR** = clicks / impressions
   - **CPA** = spend / conversions
   - **CPM** = (spend / impressions) × 1,000
4. **UNIONs** all three platforms into one queryable table (35 columns)

---

## How to Run

### Option 1 — BigQuery (Full Pipeline)

```
1. Upload raw CSVs as BigQuery tables:
   marketing_data.facebook_ads
   marketing_data.google_ads
   marketing_data.tiktok_ads

2. Run SQL files in order:
   unified_model.sql
   views/01_platform_summary.sql
   views/02_campaign_performance.sql
   ... (remaining views)

3. Connect each view as a data source in Looker Studio
   (see LOOKER_STUDIO_SETUP.md for full instructions)
```

### Option 2 — Local Python Validation

```bash
pip install pandas
python analysis.py
```

Mirrors all BigQuery views locally — use this to verify numbers before deploying to BigQuery.

### Option 3 — Instant HTML Dashboard

Open `dashboard.html` in any browser. No server or dependencies required.

---

## Key Findings

### 1. Facebook delivers the best return on spend

| Platform | Budget Share | Conversion Share | CPA |
|---|---|---|---|
| **Facebook** | 14% | **17.9%** | **$7.64** |
| Google | 28.9% | 31.6% | $8.93 |
| TikTok | 57.0% | 50.5% | $11.00 |

Facebook converts more than it costs. TikTok is the opposite — 57% of budget for only 50.5% of conversions.

### 2. Google Search Brand Terms is the top-performing campaign

| Rank | Platform | Campaign | CPA | CTR |
|---|---|---|---|---|
| 1 | Google | Search_Brand_Terms | $5.10 | 5.22% |
| 2 | Facebook | Conversions_Retargeting | $5.95 | 4.63% |
| 3 | Google | Shopping_All_Products | $6.34 | 3.34% |

Worst performer: Google Search_Generic_Terms at **$24.80 CPA** — 4.9× worse than brand terms.

### 3. TikTok's video funnel drops sharply after the first 25%

| Checkpoint | Viewers Reached |
|---|---|
| 25% | 78.2% |
| 50% | 57.4% |
| 75% | 39.7% |
| 100% (completed) | **26.0%** |

Only 1 in 4 viewers watches a TikTok ad to completion.

### 4. Facebook CPA is improving week over week

| Week | Facebook CPA | Google CPA | TikTok CPA |
|---|---|---|---|
| Week 1 | $7.83 | $8.65 | $11.28 |
| Week 5 | **$7.39** | $9.08 | $10.91 |

Facebook CPA improved **5.6%** over 5 weeks — algorithm learning in action.

### 5. Google Quality Score strongly predicts CTR

| Quality Score | Avg CTR |
|---|---|
| 7 | 1.09% |
| 8 | 3.33% |
| 9 | **5.21%** |

QS 9 campaigns deliver **2.6× higher CTR** than QS 7.

---

## Unified Schema (35 columns)

```
date, platform, campaign_id, campaign_name,
ad_group_id, ad_group_name, impressions, clicks, spend,
conversions, video_views,
ctr, cpa, cpm, cpc, conversion_rate, roas,
video_completion_rate,
engagement_rate, reach, frequency,              -- Facebook
conversion_value, avg_cpc, quality_score,
search_impression_share,                        -- Google
video_watch_25/50/75/100, likes, shares,
comments, total_engagements                     -- TikTok
```

---

## Assumptions

- All monetary values are in USD
- `conversions` represents the same event type across platforms
- January 2024 data is representative; no major mid-month campaign changes
- Google's native CTR is overridden by the calculated value for cross-platform consistency
- NULL values in platform-specific columns mean "not applicable", not missing data

---

## Tech Stack

| Tool | Purpose |
|---|---|
| BigQuery Standard SQL | Data normalization, KPI calculation, views |
| Python (pandas) | Local validation of SQL logic |
| Looker Studio | Dashboard and visualization |
| HTML/JS | Self-contained dashboard alternative |

---

## Author

Built as part of the **Improvado Marketing Analyst Technical Assignment**.
