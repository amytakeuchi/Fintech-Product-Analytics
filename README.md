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

```text
                     CUSTOMER BEHAVIOR
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Recency        Frequency       Monetary
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                   BEHAVIORAL SEGMENTS
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           Loyal         At-Risk       Occasional
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                    RETENTION OUTCOMES
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Retained       Reactivated      Churned
                            │
                            ▼
                     LIFECYCLE ACTIONS
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Protect        Win-back        Activate
```

---

# Key Hypotheses
## 1. Retention & engagement diagnosis
### H1 — Engagement vs. Customer Value

> **Purchase recency and frequency are stronger predictors of future churn than transaction value, suggesting that habitual engagement may be more important for retention than simply spending more.**

**Finding:**

- Against Recency, churn probability increases materially between 90-120 days and 121-180 days while higher frequency shows modest impact on churn probability with 20% Churn rate on 50+ more frequent buyers.
- Recency shows string correlation with Churn, having 0.84 Pearson and 0.61 Spearman. On the other hand, Frequency has negative correlation between Churn with -0.11 Pearson and -0.16 Spearman.
- It is important to note that the definition of “Churn” here is “no transaction from 90 days before the cutoff of the data” and this can explain hte strong correlation between recency.

**Why this matters:** A high-spending customer is not necessarily a loyal customer. A customer who purchases frequently may provide a stronger signal of an established behavioral habit.

**Analysis:**
- RFM feature engineering
- Pearson/Spearman correlation analysis
- Churn-rate comparison across engagement tiers
- Logistic regression / predictive modeling
- Feature importance and model interpretation

**Business question:** Should lifecycle teams prioritize behavioral engagement signals over monetary value when identifying customers at risk?

---

### H2 — Inactivity & Reactivation

> **Among cards observed to still be inactive at day 10, the probability of reactivating within the next N days is materially lower than for cards observed to still be inactive at day 5. (Day 30 will be examined as a secondary/extended landmark to check whether the effect strengthens further out.)**

**Finding:**
-

**Why this matters:** A lifecycle team needs to know when inactivity becomes actionable. Rather than arbitrarily defining 30 days as "inactive," this analysis treats inactivity as a **time-to-reactivation problem**.

```text
Last transaction
       │
       ▼
   INACTIVITY
       │
       ├──────────────► Subsequent transaction
       │                      ↓
       │                  Reactivated
       │
       └──────────────► Observation ends
                              ↓
                         Right-censored
```

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

**Business question:** At what point does waiting for organic reactivation become less attractive than initiating a win-back intervention?

---

### H3 — Cohort Retention & Activation Risk

> **Do newer customer cohorts exhibit steeper early retention declines than older cohorts, indicating an onboarding or activation opportunity?**

**Why this matters:** Overall retention can hide differences in customer lifecycle maturity. A recently activated customer may behave very differently from a customer who has been using the card for months.

**Analysis:**
- Cohort construction using `first_active_month`
- Cohort retention tables
- Month-over-month retention
- Retention drop-off analysis
- Cohort heatmaps
- Comparison of early lifecycle curves

**Business question:** Are newer customers failing to develop repeat usage at the same rate as established customers?

If supported, this would suggest that the growth/lifecycle team should focus on early activation and habit formation, rather than relying primarily on later-stage win-back campaigns.

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

## Customer Value Concentration

Initial RFM analysis shows a meaningful concentration of transaction value among higher-value customers.

> **The top 36.19% of users (Top + High tiers) account for 59.62% of total monetary value.**

This suggests that customer value is concentrated among a relatively smaller group of users, making retention of high-value customers particularly important.

The analysis therefore goes beyond identifying high-value customers and asks: **Which high-value customers are currently healthy, and which are showing behavioral signs of future risk?**

---

### H4 — Behavioral Segmentation & Retention

> **Distinct behavioral segments exist, based on RFM and engagement characteristics, and these segments exhibit meaningfully different retention curves — not merely different spending levels.**

**Why this matters:** Traditional RFM segmentation can identify customers who spend more or transact more frequently. The more important product question is whether these behavioral differences actually correspond to different lifecycle outcomes.

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

### H5 — Merchant & Category Diversity as a Loyalty Signal

> **Do customers who transact across multiple merchants/categories demonstrate higher retention than single-merchant customers, even after accounting for transaction frequency?**

**Why this matters:** Frequency alone may not capture the depth of a customer's relationship with the payment network. A customer who uses the card across restaurants, retail, travel, groceries, and entertainment may have more opportunities and reasons to continue using the card than someone whose usage is concentrated at a single merchant.

**Variables:** `unique_merchant_count`, `category_diversity`, `frequency`, `recency`, `monetary`, retention/churn outcome

**Methodology:**
1. Compare retention between low- and high-diversity users.
2. Compare diversity across frequency levels.
3. Model retention/churn while controlling for frequency.
4. Evaluate whether diversity remains associated with retention.

**Business question:** Is breadth of engagement itself associated with loyalty, or is it simply a consequence of being a frequent user?

---

### H6 — Validating Anonymized Behavioral Segments

> **Do combinations of anonymized `feature_1`, `feature_2`, and `feature_3` correspond to statistically different engagement patterns?**

**Why this matters:** Real product analytics teams frequently receive predefined customer attributes or acquisition segments. An analyst should not simply report that the segments are different — the question is whether the behavioral differences are statistically meaningful, and which engagement dimensions actually differ.

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
