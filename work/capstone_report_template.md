# Capstone Report — Machine Learning

- **Author:** Mohamed Ahmed
- **Lane:** Machine Learning
- **Repo:** https://github.com/mohamedahmed02/Flyrank-internship
- **Date:** August 2026

---

## 0. Abstract

This study asks whether a simple machine-learning ranking model can help prioritize content items for review using signals available before a future performance period.

The analysis uses 158,549 content-item observations built from public-safe March 2026 search and engagement features, with April 2026 organic clicks used to define the future outcome.

A Logistic Regression model using five search and engagement features was evaluated with a client-grouped train/test split so that clients did not overlap between training and testing.

On the held-out test split, the model achieved 0.3800 Precision@50 compared with 0.1200 for the Week-4 rule baseline, while the test positive rate was 0.1641.

The result is best interpreted as directional decision-support evidence for prioritizing human content review, not as evidence that the model or a content refresh causes future traffic growth.

---

## 1. Problem framing

The practical problem is deciding which content items should receive review attention first when a content inventory is too large to inspect manually.

The unit of analysis is a content item identified internally using hashed client and content identifiers.

The model produces a ranking score for each content item. The highest-ranked items can then be placed near the top of a human review queue.

The supported decision is therefore:

> Which content items should enter the review queue first?

A wrong call can waste editorial time by prioritizing an item that does not show the expected future outcome, while a missed item may delay attention to content that could have been worth reviewing.

A rule-based baseline provides a simple prioritization mechanism, but machine learning can combine several available signals into one ranking score. The goal of this analysis is to test whether that ranking provides better top-of-queue concentration than the existing rule baseline.

The model is used as decision support rather than as an automated content-refresh decision maker.

---

## 2. Data safety

The modeling dataset contains public-safe aggregated content-item observations from the FlyRank ML Internship warehouse release.

The analysis uses:

- March 2026 search and engagement signals as decision-time features.
- April 2026 organic clicks as the future outcome used to construct the label.
- Hashed client identifiers for grouped validation only.
- Hashed content identifiers for joining observations across the two monthly periods.

The five modeling features are:

1. `gsc_clicks`
2. `gsc_impressions`
3. `gsc_avg_position`
4. `ga4_sessions`
5. `scroll_events`

The following information was deliberately excluded from the model:

- `april_clicks`, because it is future information and is used only to construct the label.
- `label`, because it is the target itself.
- Hashed client identifiers, because they are used only to create client-grouped validation splits.
- Hashed content identifiers, because identifiers are not treated as meaningful predictive features.
- Any client names, content URLs, private search queries, or other identifying business information.

The label-derived fields `trend_direction` and `trend_pct` were also treated as leakage risks and were not used as modeling features.

The target is defined as:

> A content item receives a positive label when April organic clicks are greater than March organic clicks.

The final modeling dataset contains 158,549 observations.

The overall positive rate is 0.1762, or 17.62%.

GA4 sessions and scroll events are missing for 44,696 observations each. These missing values were retained and handled in the modeling pipeline rather than silently interpreting missingness as observed activity.

The public-facing analysis contains no client names, private queries, content URLs, or other identifying business information.

---

## 3. Baseline

The first prioritization approach was the Week-4 rule-based baseline.

The baseline produces a transparent score and ranks content items using the predefined rule rather than a learned model.

It is a fair comparison because the baseline and Logistic Regression model are evaluated on the same held-out test observations using the same metric: Precision@50.

The final grouped test set contains 21,104 observations with a positive rate of 0.1641.

The measured results were:

| Method | Precision@50 |
|---|---:|
| Week-4 rule baseline | 0.1200 |
| Logistic Regression | 0.3800 |

The baseline therefore provides a transparent reference point for judging whether the learned ranking adds useful prioritization signal.

---

## 4. Model / analysis

A Logistic Regression model was selected as the primary model because it is lightweight, interpretable, and appropriate for producing a probability-based ranking score.

The model uses exactly five March 2026 features:

- Google Search Console clicks
- Google Search Console impressions
- Google Search Console average position
- GA4 sessions
- Scroll events

The model produces a probability score for the positive class. Content items are ranked from the highest predicted probability to the lowest predicted probability.

The target is binary:

> `label = 1` when April organic clicks are greater than March organic clicks; otherwise `label = 0`.

Future April clicks are not provided to the model as features.

The purpose of the model is not to predict a search-engine algorithm or guarantee traffic growth. Its purpose is to provide a ranking signal for content review prioritization.

---

## 5. Evaluation

### Validation design

The evaluation uses a grouped train/test split by client.

This design was chosen to prevent observations belonging to the same client from appearing in both training and testing.

The final split contains:

| Evaluation property | Value |
|---|---:|
| Train rows | 137,445 |
| Test rows | 21,104 |
| Train clients | 36 |
| Test clients | 10 |
| Client overlap | 0 |

The zero client overlap confirms that the test clients were not represented in the training set.

### Metric

The primary evaluation metric is Precision@50.

Precision@50 measures the proportion of positive outcomes among the 50 highest-ranked content items.

The test positive rate was 0.1641, meaning approximately 16.41% of test observations were positive.

### Model vs baseline

| Method | Precision@50 | Test positive rate |
|---|---:|---:|
| Week-4 rule baseline | 0.1200 | 0.1641 |
| Logistic Regression | 0.3800 | 0.1641 |

The Logistic Regression model achieved 0.3800 Precision@50 compared with 0.1200 for the baseline on the same grouped test split.

This corresponds to a 0.26 absolute Precision@50 improvement over the baseline.

### Error analysis

The ranked predictions show that the model does not perfectly separate positive and negative outcomes. For example, some high-scoring observations in the ranked queue have a negative label.

This is expected for a probabilistic ranking model and reinforces that the score should be used to prioritize human review rather than as an automatic decision rule.

The top-ranked queue therefore represents a concentration of positive outcomes rather than a guaranteed list of successful content refresh opportunities.

### Leakage and sanity checks

The evaluation included explicit checks for:

- Zero client overlap between training and testing.
- Future April clicks being used only for label construction.
- Required modeling features being present.
- Consistent test-set size.
- Consistent test positive rate.
- Model and baseline being evaluated on the same split.
- The reported Precision@50 values matching the final evaluation.

---

## 6. Interpretation

The main observed result is that the Logistic Regression ranking concentrated more positive outcomes in its top 50 items than the Week-4 rule baseline on the held-out grouped test set.

The measured Precision@50 values were:

- Week-4 baseline: 0.1200
- Logistic Regression: 0.3800
- Test positive rate: 0.1641

The model therefore provided a stronger top-of-queue ranking signal than the rule baseline in this evaluation.

The model uses search visibility and engagement signals together rather than relying on a single rule. Higher-ranked items should be interpreted as items that the model considers more likely to match the defined future-outcome pattern.

An important observation is that high model probability does not guarantee a positive outcome. The ranked prediction sample includes high-scoring observations with label 0.

This means the model is useful for prioritization, but not sufficiently reliable to replace human review.

The result is directional evidence from the evaluated data rather than proof of causality.

---

## 7. Recommendation

### 1. Prioritize the model's highest-ranked items for human review

Use the model score to order the review queue and start with the highest-ranked content items.

**Confidence:** Moderate for prioritization within the evaluated setting.

The evidence supports improved top-50 concentration compared with the Week-4 baseline, but the result is limited to the evaluated test split.

### 2. Review search visibility and positioning context

For highly ranked items, inspect impressions, clicks, and average search position before deciding whether a refresh is appropriate.

The model score should identify candidates for investigation, not automatically trigger an editorial change.

### 3. Use engagement signals as supporting evidence

Where available, GA4 sessions and scroll events can provide additional context.

Because these fields are missing for a substantial portion of observations, they should not be treated as universally available signals.

### 4. Keep human review in the loop

Before changing content, review:

- Search intent
- Content quality
- Relevance
- Freshness
- Existing page context
- Editorial considerations

The model should support these decisions rather than replace them.

### 5. Measure post-intervention outcomes

If a content refresh is performed, track the subsequent outcome separately.

Future evaluation periods should be used to determine whether the prioritization strategy remains useful over time.

### Recommended workflow

> **Rank → Review → Refresh if justified → Measure → Learn**

The strongest supported claim is that the evaluated Logistic Regression model provided better top-50 ranking precision than the Week-4 rule baseline on the held-out grouped test split.

It should not be interpreted as evidence that using the model or refreshing a page causes future organic traffic growth.

---

## 8. Reproducibility

The project repository contains the capstone notebook and supporting artifacts used for the analysis.

Repository:

https://github.com/mohamedahmed02/Flyrank-internship

Capstone notebook:

https://github.com/mohamedahmed02/Flyrank-internship/blob/main/work/notebooks/capstone.ipynb

The main analysis is documented in:

`work/notebooks/capstone.ipynb`

The notebook includes the data preparation, feature checks, grouped validation, model training, baseline comparison, evaluation sanity checks, and ranked outputs.

### Reproduction configuration

The analysis uses:

- Python
- pandas
- NumPy
- scikit-learn
- Logistic Regression
- Grouped client validation

The evaluation split is grouped by client, with:

- 36 training clients
- 10 test clients
- 0 client overlap

The reported evaluation uses:

- 21,104 test rows
- 0.1641 test positive rate
- Precision@50 as the primary metric
- Week-4 rule baseline = 0.1200
- Logistic Regression = 0.3800

The public-facing paper is available at:

https://mohamedahmed02.github.io/Flyrank-internship/

The project intentionally does not publish private client information, content URLs, private queries, or identifying business information.

---

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset.

Data source: https://flyrank.ai

This work was completed as part of the FlyRank Machine Learning Internship track and is presented as a public-safe research and decision-support artifact.

---

## Claims checklist

The report uses careful claim language such as:

- observed
- measured
- directional
- decision-support

The results are not presented as causal evidence.

The model does not claim to predict Google's algorithm.

The analysis does not use client-identifying information as model features.

Precision@50 is reported alongside the test positive rate.

The reported model and baseline metrics were produced on the same held-out grouped test split.

The final result is:

> **Logistic Regression Precision@50 = 0.3800 vs Week-4 baseline Precision@50 = 0.1200, with a test positive rate of 0.1641.**
