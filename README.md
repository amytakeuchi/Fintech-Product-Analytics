# Fintech-Product-Analytics
# Customer Retention & Behavioral Segmentation Analytics

## Executive Summary

**How can transaction behavior be used to identify customer lifecycle risk, understand retention, and inform targeted engagement strategies?**

This project analyzes transaction-level data from **Elo**, a Brazilian payment network, to understand how cardholder behavior relates to **engagement, reactivation, churn, customer value, and retention**.

Rather than treating customer value as simply a spending problem, the analysis investigates whether **behavioral habits — recency, frequency, merchant/category breadth, and transaction patterns — provide earlier and more actionable signals of customer retention risk.**

### Analytical Approach

The project combines:

- Customer-level RFM analysis
- Retention & churn analysis
- Time-to-reactivation / survival analysis
- Behavioral segmentation & clustering
- Cohort retention analysis
- Statistical hypothesis testing
- Lifecycle and win-back strategy recommendations

> **Core business question:**
> *Can we identify distinct customer behaviors and detect declining engagement early enough to enable differentiated lifecycle intervention?*

---

## Business Context

Elo operates a payment network and partners with merchants to provide cardholders with relevant offers and promotions.

The underlying business challenge is not simply predicting how much a customer will spend. A useful customer analytics system should help answer:

1. Who is engaged and who is becoming inactive?
2. How long can a customer remain inactive before reactivation becomes unlikely?
3. Which behavioral segments have fundamentally different retention patterns?
4. Are high-value customers always loyal, or can valuable customers also become at risk?
5. Can customer behavior be translated into actionable lifecycle segments?

This project approaches the problem from a **product analytics and lifecycle management perspective**, moving from:

**Behavioral measurement → Diagnosis → Segmentation → Retention outcomes → Potential intervention**

---

## Analytical Framework
<img src="image/readme_diagram.png" title="Cohort Dropoff Visualization">

---

# Key Hypotheses
## 1. Retention & engagement diagnosis
### H1 — Engagement (Recency & Frequency) vs. Customer Value --> Churn

> **Purchase recency and frequency are stronger predictors of future churn than transaction value, suggesting that habitual engagement may be more important for retention than simply spending more.** (**Business question:** Should lifecycle teams prioritize behavioral engagement signals over monetary value when identifying customers at risk?)

**Finding:**

- Against Recency, churn probability increases materially between 90-120 days and 121-180 days while higher frequency shows modest impact on churn probability with 20% Churn rate on 50+ more frequent buyers.
- Recency shows strong correlation with Churn, having 0.84 Pearson and 0.61 Spearman. On the other hand, Frequency has negative correlation between Churn with -0.11 Pearson and -0.16 Spearman.
- It is important to note that the definition of “Churn” here is “no transaction from 90 days before the cutoff of the data” and this can explain hte strong correlation between recency.

<img src="image/rf_churn.png" width="1000" title="Recency & Frequency vs Churn rate">

**Why this matters:** A high-spending customer is not necessarily a loyal customer. A customer who purchases frequently may provide a stronger signal of an established behavioral habit.

**Analysis:**
- RFM feature engineering
- Pearson/Spearman correlation analysis
- Churn-rate comparison across engagement tiers
- Logistic regression / predictive modeling
- Feature importance and model interpretation

---

### H2 — D5 vs D10 Retention --> Inactivity & Reactivation 

> **The longer a card stays inactive, the less likely it is to come back.** Cards still inactive at day 10 reactivate slower than cards inactive at day 5 — and day 30 is checked as a further point to see if the effect keeps compounding. (**Business question:** At what point does waiting for organic reactivation become less attractive than initiating a win-back intervention?)

**Finding:**
- Cards inactive for 30+ days show a statistically significant difference in subsequent reactivation compared to cards inactive for only 5–10 days (log-rank p < 0.005).
- The practical difference is large: median time-to-reactivation is 6.08 days for day-5 survivors/9.68 days for day-10 survivors vs. 16.26 days for day-30 survivors

**Methodology:**
- Transaction-gap analysis
- Distribution of time-to-reactivation
- Right-censoring
- Kaplan-Meier survival analysis
- Reactivation probability at 7/14/21/30/45/60/90 days
- Evaluation of the 30-day candidate threshold

**Initial finding** — among completed reactivation episodes:

| Percentile | Gap |
|---|---:|
| 90th | 7.1 days |
| 95th | 14.6 days |
| 97th | 21.8 days |
| 98th | 27.8 days |
| 99th | 37.8 days |

The 98th percentile is approximately 28 days, making 30 days an interesting candidate threshold for further investigation — not an assumption that 30 days is automatically correct.

---

### H3 — Cohort Retention & Activation Risk (Earlier Cohort --> Retention Decline)

> **Do newer customer cohorts exhibit steeper early retention declines than older cohorts, indicating an onboarding or activation opportunity?** (**Business question:** Are newer customers failing to develop repeat usage at the same rate as established customers?)

**Finding:**
- Earlier cohorts in February and March exhibit steeper earlier drop off with -19.4, -20.1, and -17.1 dropoffs in the first month and -2.8, -0..8, and -1.0 in the subsequent month
- --> the growth/lifecycle team should focus on early activation and habit formation, rather than lying to much on primarily on later-stage win-back campaigns.
- Later win-back campaigns still matters as each cohort still see increase in drop-off especially at the beginning of the year.


<img src="image/cohort_dropoff.png" width="800" title="Cohort Dropoff Visualization">

---

## RFM & Customer Value Analysis

### Customer-Level Behavioral Features

RFM provides the foundation for customer behavioral analysis.

**Recency** — How recently did the customer transact?
```text
Recency = observation_date − last_purchase_date
```
Lower recency indicates more recent engagement.

**Frequency** — How often did the customer transact?
```text
Frequency = number of transactions
```
Frequency is used as a proxy for behavioral engagement and habit formation.

**Monetary** — How much did the customer spend?
```text
Monetary = total transaction value
```

**Additional behavioral measures:**
- Average transaction value / AOV
- Unique merchant count
- Category diversity
- Average transaction gap
- Transactions per active month

---

## [Monetary] Customer Value Concentration Analysis 

Initial RFM analysis shows a meaningful concentration of transaction value among higher-value customers.

**Findings**
- The top 36.19% of users (Top + High tier) account for 59.62% of total monetary value.
- The remaining 44.05% of users (Mid-high + Mid tier) account for 39.07% of total monetary value.
- This suggests that customer value is concentrated among a relatively smaller group of users, making retention of high-value customers particularly important.

The analysis therefore goes beyond identifying high-value customers and asks: **Which high-value customers are currently healthy, and which are showing behavioral signs of future risk?**

---

## Frequency Analysis

Purchase frequency is a stronger predictor of total customer value (LTV), while it doesn't have any impact after controlling average order value (AOV).
**Findings**
- Spearman ρ = 0.930 → strong monotonic link (marginal, ignores other vars)
- OLS std coef = 0.007 → almost no unique effect **once AOV is controlled**
- Why Spearman still matters:
- Monetary is heavily skewed (skew=64, kurtosis=5108) → OLS assumptions (normal residuals, linear fit) are badly violated
- Skewed data → Pearson/OLS coefficients get distorted by outliers; Spearman (rank-based) is robust to that Spearman correctly shows frequency is associated with monetary — OLS just shows that association is mediated through AOV, not independent

---

### H4 — Behavioral Segmentation & Retention (Behavioral Segment --> Retention)

> **Distinct behavioral segments exist, based on RFM and engagement characteristics, and these segments exhibit meaningfully different retention curves — not merely different spending levels.**

**Methodology:**

1. **Create customer-level behavioral features** — candidate segmentation variables:
   - `recency_days`
   - `frequency`
   - `monetary`
   - `avg_transaction_amount`
   - `unique_merchant_count`
   - `category_diversity`
   - `avg_days_between_transactions`

2. **Transform and standardize** (transaction behavior is highly skewed):
   ```text
   Raw Features → log1p Transformation → StandardScaler → Clustering
   ```

3. **Identify behavioral clusters** — K-Means clustering evaluated across multiple values of `k`, considering silhouette score, cluster size, behavioral separation, and business interpretability.

4. **Profile each segment** by recency, frequency, monetary, merchant diversity, category diversity, and transaction behavior.

5. **Validate segments against retention** — the key test is whether segments have different post-baseline retention curves, using Kaplan-Meier retention curves, log-rank tests, 30/60/90-day retention, and optional Cox proportional hazards modeling.

**Business question:** Do behavioral segments represent genuinely different lifecycle states that warrant different retention strategies?

---

### H5 — Merchant & Category Diversity as a Loyalty Signal (Category/Merchant Diversity Segments --> Retention)

> **Do customers who transact across multiple merchants/categories demonstrate higher retention than single-merchant customers, even after accounting for transaction frequency?**

**Variables:** `unique_merchant_count`, `category_diversity`, `frequency`, `recency`, `monetary`, retention/churn outcome

**Methodology:**
1. Compare retention between low- and high-diversity users.
2. Compare diversity across frequency levels.
3. Model retention/churn while controlling for frequency.
4. Evaluate whether diversity remains associated with retention.

**Business question:** Is breadth of engagement itself associated with loyalty, or is it simply a consequence of being a frequent user?

---

### H6 — Validating Anonymized Behavioral Segments (Features --> Engagement)

> **Do combinations of anonymized `feature_1`, `feature_2`, and `feature_3` correspond to statistically different engagement patterns?**

**Methodology:**
- Group customers by `feature_1`, `feature_2`, `feature_3` combinations
- Compare behavioral measures
- MANOVA for multivariate engagement differences
- Follow-up univariate tests where appropriate
- Effect-size interpretation
- Business interpretation of meaningful segment differences

**Business question:** Can we validate whether an existing segmentation framework actually corresponds to different customer behaviors?

---

## Retention & Churn Definition

Because the dataset contains a finite transaction observation window, churn must account for incomplete follow-up.

### Observed Churn

For a fixed 90-day churn definition:

> A card is classified as having observed churn when its last observed transaction occurred at least 90 days before the end of the available transaction observation period.

```text
Last transaction
       │
       ├──────────── 90+ days ────────────► Observation end
       │
       ▼
  Observed churn
```

### Right-Censored Customers

If the last transaction occurred less than 90 days before the observation period ended:

> The card is right-censored because there is insufficient follow-up to determine whether it would eventually reach 90 days of inactivity.

```text
Last transaction
       │
       ├──────── 30 days ────────► Data ends
       │
       ▼
 Right-censored
```

This should **not** be interpreted as either churned or retained.

**Why survival analysis is useful:** For analyses involving time-to-event outcomes, right-censored observations can be incorporated using survival-analysis techniques rather than incorrectly assigning them a binary churn outcome.

---

## Dataset

The project uses transaction and card-level information from the Elo customer loyalty dataset.

| Dataset | Purpose |
|---|---|
| `train.csv` | Card-level information and loyalty target |
| `test.csv` | Card-level information |
| `historical_transactions.csv` | Historical transaction behavior |
| `new_merchant_transactions.csv` | Transactions at merchants not previously visited by the card |
| `merchants.csv` | Merchant-level attributes |
| `Data_Dictionary.xlsx` | Data field definitions |

> **Data note:** The dataset is simulated/fictitious and does not represent real customer data.

### Important Transaction-Data Consideration

`new_merchant_transactions.csv` contains transactions at **new-to-card merchants**, rather than representing a complete transaction history for the period. Therefore, absence of a transaction in this dataset cannot automatically be interpreted as customer inactivity or churn.

For analyses requiring complete transaction continuity — particularly reactivation, retention, and churn — the transaction coverage and observation window are explicitly considered when defining outcomes.

---

## Analytical Methodology

**Descriptive Analytics**
- RFM analysis
- Distribution analysis
- Customer value concentration
- Transaction-gap analysis
- Cohort analysis

**Statistical Analysis**
- Pearson/Spearman correlation
- Group comparisons
- Hypothesis testing
- MANOVA
- Log-rank tests
- Effect sizes

**Survival Analysis**
- Time-to-reactivation
- Kaplan-Meier curves
- Right-censoring
- Retention curves
- Optional Cox proportional hazards models

**Machine Learning**
- K-Means clustering
- Customer segmentation
- Logistic regression
- Model interpretation

---

## From Analysis to Lifecycle Strategy

The ultimate goal is not simply to produce statistically significant findings — the analysis is designed to translate behavioral signals into actionable lifecycle treatment.

| Behavioral State | Potential Lifecycle Objective |
|---|---|
| High-frequency / high-value | Protect & reward loyalty |
| High-value + declining recency | Prevent revenue loss |
| Recently activated + low frequency | Build habit / activation |
| Long inactivity | Win-back intervention |
| Low engagement | Re-engagement or lower-cost treatment |
| High merchant/category diversity | Protect broad card usage |

The exact treatment should be determined by the magnitude and statistical evidence from the analysis.

---

## Key Deliverables

1. **Customer Behavioral Profile** — RFM-based view of who generates value, who is highly engaged, and where customer value is concentrated.
2. **Churn & Reactivation Framework** — a lifecycle framework distinguishing Active → Increasing inactivity → Potential intervention → Long-term inactivity → Observed churn.
3. **Retention Curves** — comparison of retention trajectories across cohorts, behavioral segments, and engagement levels.
4. **Behavioral Segmentation** — customer clusters characterized by engagement, value, recency, frequency, merchant breadth, and category breadth.
5. **Actionable Lifecycle Insights** — activation opportunities, win-back thresholds, high-value customer protection, and segment-specific lifecycle strategies.

---

## Project Structure

```text
elo-customer-retention/
│
├── data/
│   └── README.md
│
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_rfm_analysis.ipynb
│   ├── 03_retention_churn.ipynb
│   ├── 04_reactivation_survival.ipynb
│   ├── 05_behavioral_segmentation.ipynb
│   └── 06_hypothesis_testing.ipynb
│
├── src/
│   ├── preprocessing.py
│   ├── features.py
│   ├── segmentation.py
│   └── retention.py
│
├── outputs/
│   ├── figures/
│   └── tables/
│
├── README.md
└── requirements.txt
```

---

## What This Project Demonstrates

This project demonstrates an end-to-end product/customer analytics workflow:

```text
Raw transaction data
        ↓
Data preparation
        ↓
Customer behavioral features
        ↓
RFM & value analysis
        ↓
Retention / churn diagnosis
        ↓
Reactivation & survival analysis
        ↓
Behavioral segmentation
        ↓
Statistical validation
        ↓
Lifecycle recommendations
```

Rather than treating analytics as a collection of dashboards or models, the project focuses on connecting:

**Customer behavior → Measurable outcomes → Statistical evidence → Product/lifecycle decisions**

---

## Final Objective

> **Identify the behavioral signals and customer segments that matter most for retention, determine when declining engagement becomes actionable, and translate those findings into differentiated lifecycle strategies.**
