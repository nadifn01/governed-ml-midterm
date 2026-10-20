# BANA 690A Midterm: Written Answers

**Name:** Farha Nadine
**Last-name group (A to H, I to P, Q to Z):** I to P
**GitHub username:** nadifn01

This file is half of your exam. The code and the pipeline grade the other half.
Write in full sentences. Cite evidence from THIS repository: a file name, a line,
a number from `artifacts/gate_report.json` or `artifacts/model_card.md`.
Open book. Individual. Keep each answer short and specific.

---

## Lab 1: Make the data compliant

**1.1** Which four tests failed when you first ran `python -m pytest -q tests/test_lab1_compliance.py`? For each, name the control type (list or detective).

These are 4 tests fails that is test_no_zip_like_column, test_no_date_like_column, test_no_suspiciously_predictive_column, and test_policy_agrees_with_data_dictionary are detective controls. Inspect raw values, distributions, and schema attributes to identify risks. The controls that passed are preventive controls enforcing exclusion rules outlined in policy configuration.

**1.2** What did you add to `governance/policy.json`, and under which keys?
Added zip and date_of_birth to pii_columns (quasi-identifiers); added adjuster_fraud_review to leakage_columns (outcome recorded post-decision)

**1.3** Why is the fix in the policy and not in `src/prepare.py`? One or two sentences.
Updating policy.json preserves governance and auditability, while editing pipeline code bypasses these controls. This separation ensures oversight without changing code.
---

## Lab 2: Build features from driving history

Record every feature you implemented. Four minimum, four families minimum, including both families assigned to your last-name group.

| Feature name | Family | Formula in one sentence | Why it should predict claim_free |
|---|---|---|---|
| claims_per_month | Frequency | total claims/24 | Measures baseline event frequency per exposure unit.
| speeding_velocity | Velocity | rate recent/(rate overall +0.1) | Captures sudden behavioral acceleration or reckless habit spikes
| hard_brake_trend | Trend | rate m 13-24 −rate m1-12| Measures deterioration or improvement in vehicle handling over time.
| max_hard_brakes_per_1k | Extremes | max i (brakes i/miles i ×1000) | Detects outlier acute hazard spikes that aggregate averages hide. 
| night_share_mean | Ratio | mean(night_miles_share) | Predicts risk exposure related to low-visibility driving windows.


**2.1** Explain the temporal fence test in your own words. What would have happened to your model in production if a feature had failed it?

The temporal fence ensures only data from months ≤24 is used, preventing data leakage from future outcomes and avoiding discrepancies between offline and live performance.

---

## Lab 3: Hunt the leakage

Three suspect columns. For each, give your ruling and the evidence.

| Column | Ruling (leakage or allowed) | Correlation with label | When recorded | Reasoning |
|---|---|---|---|---|


**3.1** One of these columns is leakage even though it did not trip the correlation test. Why can a numeric test miss it, and what kind of test catches it?
The extreme gap in missing values (96% vs. 50%) proves renewal decisions depend directly on claims outcomes.
It happens at policy year-end, long after underwriting decisions are made. A model cannot use information at prediction time that does not exist yet.
---


## Lab 4: Run v1 and v2, explain the block

**4.1** Paste the DECISION line and overall AUC for v1 and for v2.
v1: Overall AUC 0.7046; Rural approval 0.7749, Urban approval 0.8951; Disparity ratio 0.8657. Gate Decision: PASS.
v2: Overall AUC 0.7224; Rural approval 0.5146, Urban approval 0.9196; Disparity ratio 0.5596. Gate Decision: BLOCKED (Fails subgroup_auc[rural], subgroup_tpr[rural], approval_disparity_ratio, tpr_ratio).

**4.2** v2 has the higher AUC. Why is it blocked? Point to the exact line in `data/generate_data.py` that creates `territory_risk_index`, and the line that creates the label's rural penalty.
Line 1: territory_risk_index = np.clip(42 + 30 * rural + ..., 0, 100)
Generates a synthetic score mechanically skewed 30 points higher purely by rural status, acting as an uncalibrated proxy for geographic area.
Line 2: logit = 1.35 + z - 1.10 * rural + rng.normal(0, 1.05, n)
Directly penalizes rural applicants by deducting 1.10 in the log-odds calculation for claim_free, encoding historical disparate treatment into the ground-truth target.

**4.3** v2 passed `no_prohibited_features`. Explain why a preventive list check cannot catch a proxy, and name the detective check that did.
Preventive controls like no_prohibited_features rely on column names and can miss proxies like territory_risk_index. Detective checks assess outcome disparities across subgroups, catching proxy leakage regardless of feature names.

---

## Lab 5: Change the governance

**5.1** What new policy key did you add, and what value did you choose?

Added "min_fpr_ratio": 0.75; updated "policy_version" to "1.1"

**5.2** Defend the value in two or three sentences. What does a false positive mean for Sierra Mutual, and for the driver? Why is unequal FPR across urban and rural drivers a fairness problem, not only a profit problem?

Unequal false positive rates (FPR) create unfair standards, causing financial losses from bad claims and unjust denials for qualified drivers. Enforcing both FPR and true positive rate (TPR) parity ensures equalized odds and fairness across subgroups.

**5.3** Describe the test you wrote in `tests/test_lab5_gate.py`. What does it prove that running the pipeline alone does not?

Pipeline runs test one dataset, while unit tests with synthetic data verify thresholds are enforced consistently, independent of training data changes.
---

## Lab 6: Beat the gate

**6.1** Paste your final gate table (from the terminal or `artifacts/gate_report.json`): overall AUC, disparity ratio, TPR ratio, FPR ratio, DECISION.

**6.2** Which engineered features did you use, and which one mattered most? How do you know?

**6.3** Did you try `territory_risk_index`? What happened, and what does that tell you about the difference between a better model and a deployable model?

---

## Lab 7: Request deployment

Fill in the pull request template on GitHub. Then answer here:

**7.1** The workflow paused with "awaiting approval" before it ran. Which control is that, in the language of owner, trigger and evidence?

**7.2** After approval, what did Stage A print about your governance change? Quote it.

---

## Lab 8 (bonus): Break it and watch it stop

**8.1** What did you change on your second branch, and what did `python run_pipeline.py --config configs/v2.json` print locally?

**8.2** What did Stage A print on GitHub? Quote the refusal line.

**8.3** Which control would NOT have stopped this change on its own, and why do we need both?

---

## Declaration

I completed this exam individually, using only the permitted materials.

**Signature (type your name):**
