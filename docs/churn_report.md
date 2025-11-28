
## (BA-style business report) `docs/churn_report.md`

```markdown
# Customer Churn Analysis – Business Report

## 1. Background & Objective

A telecom company is experiencing **customer churn** and wants to understand:

1. Which customers are most at risk of leaving.
2. What factors drive churn.
3. How to design targeted retention actions.

This project uses a **logistic regression model** to assign each customer a **churn probability**, and a Tableau dashboard to turn those scores into **business insights**.

---

## 2. Data Summary

- Source: Public **telco customer churn** dataset.
- Rows: ~7,000 customers.
- Target variable: `Churn` (Yes/No).

After cleaning:

- Churn encoded as:
  - **1 = churner**
  - **0 = non-churner**
- Churn rate in the dataset: roughly **26–27%** of customers.

Key feature groups:

- **Customer profile:** gender, senior citizen, dependents.
- **Services:** phone, internet, streaming TV, device protection, tech support.
- **Account / contract:** contract type, payment method, paperless billing, tenure.
- **Financials:** monthly charges, total charges.

---

## 3. Modeling Approach

### 3.1. Preprocessing

- Standardized column names to lowercase with underscores.
- Converted `TotalCharges` to numeric and removed rows with invalid values.
- Encoded `Churn` as 0/1.
- Converted categorical features to dummy variables (`pd.get_dummies`, drop_first=True).
- Split data into **training (80%)** and **test (20%)**, stratified by churn.

### 3.2. Model

- Algorithm: **Logistic Regression**
- Rationale:
  - Fast, interpretable, and works well as a baseline for classification.
  - Coefficients directly show the direction and strength of each driver.

---

## 4. Model Performance

On the **held-out test set**, the model achieved:

- **Accuracy:** ~**80%**
- **Churn class (1) metrics:**
  - Precision ≈ **65%**  
    → When the model flags a customer as high risk, it is correct ~65% of the time.
  - Recall ≈ **57%**  
    → It captures ~57% of all actual churners.
  - F1-score ≈ **0.61**
- **ROC-AUC:** ~**0.84**  
  → Strong separation between churners and non-churners.

Interpretation:

- The model is **good enough to prioritize customers by risk**.
- It is not perfect (no model is), but it provides a **useful ranking** for targeted retention campaigns.

---

## 5. Key Drivers of Churn

Using logistic regression coefficients and Tableau views, several strong drivers emerged:

### 5.1. Contract Type

- **Month-to-month contracts show the highest predicted churn probability.**
- 1-year and 2-year contracts have **much lower churn risk**.

**Business interpretation:**

- Month-to-month customers have low commitment and can leave easily.
- Longer-term contracts are associated with more loyal, stable customers.

### 5.2. Tenure (Time with Company)

- Customers with **low tenure** (e.g., < 1 year) have **much higher churn risk**.
- As tenure increases (e.g., 4–6 years, 6+ years), churn probability declines.

**Business interpretation:**

- The **first 1–2 years** are critical for customer retention.
- If customers feel unhappy early, they leave before becoming profitable long-term users.

### 5.3. Monthly Charges

- Higher **monthly charges** correlate with higher churn probabilities.

**Business interpretation:**

- Customers paying more each month may be more sensitive to:
  - Perceived value,
  - Service quality issues,
  - Competitor offers.
- High-revenue customers are therefore also **high risk** and need closer attention.

### 5.4. Payment Method

- Certain payment methods (e.g., **electronic check**) have **higher churn risk** than others (e.g., bank transfer, credit card).

**Business interpretation:**

- Payment methods may proxy for:
  - Customer digital engagement,
  - Ease of switching,
  - Demographic/behavioral differences.

---

## 6. Tableau Dashboard – Insights

Using the scored dataset (`churn_scored.csv`), the dashboard includes:

### 6.1. Churn Probability Distribution

- Histogram of predicted churn probability.
- Most customers cluster at **low risk (e.g., < 20%)**.
- There is a clear **tail of high-risk customers**, which are prime targets for retention.

### 6.2. Churn Probability by Contract

- Month-to-month: **highest average churn probability**.
- 1-year and 2-year: significantly lower average churn probability.

**Takeaway:** Move valuable month-to-month customers onto longer-term contracts where possible.

### 6.3. Churn Probability by Tenure Band

Example pattern (approximate):

- `< 1 year` and `1–2 years`: highest churn risk.
- `4–6 years`, `6+ years`: churn risk declines.

**Takeaway:** Focus retention and onboarding programs on customers in their **first 1–2 years**.

### 6.4. Churn Probability by Payment Method

- Some payment methods (e.g., electronic check) are higher risk.
- Others (e.g., automatic bank transfer/credit card) are more stable.

**Takeaway:** Encourage at-risk customers to move to more stable payment methods (e.g., auto-pay with card or bank).

## Dashboard Preview

Below is a preview of the Tableau churn dashboard built from the scored dataset:

![Customer Churn Dashboard](docs/Customer Churn Risk Overview – Telecom.png)
---

## 7. Recommended Actions

Based on the analysis, the business can take the following steps:

### 7.1. Target High-Risk Month-to-Month Customers

- **Who:** Month-to-month customers with high churn probability (e.g., > 0.6).
- **Actions:**
  - Offer **discounted upgrades** to 1-year or 2-year contracts.
  - Bundle services (internet + TV/streaming) at a promotional rate.
  - Provide loyalty rewards (points, free add-ons) for committing longer term.

### 7.2. Focus on Early-Tenure Customers

- **Who:** Customers in their **first 12–24 months**.
- **Actions:**
  - Proactive **welcome/onboarding campaigns**.
  - Check-in calls or emails at 30, 60, 90 days.
  - Satisfaction surveys and fast resolution of early issues.

### 7.3. Manage High-Charge Customers Carefully

- **Who:** High monthly charge customers with above-average churn probability.
- **Actions:**
  - Assign to a **priority support** or “premium care” queue.
  - Offer **bill reviews** to ensure they are on the best plan.
  - Provide value-focused messaging (speed, reliability, add-ons) to justify price.

### 7.4. Encourage More Stable Payment Methods

- **Who:** High-risk customers using payment methods associated with higher churn.
- **Actions:**
  - Incentivize switching to **automatic bank transfer or credit card** (e.g., bill credit, discount).
  - Highlight convenience and reduced risk of missed payments.

---

## 8. Limitations & Next Steps

**Limitations:**

- Model is based on a **historical dataset**; real-world patterns may shift over time.
- Churn is influenced by factors not captured in the data (e.g., competitor promotions, macroeconomic changes).
- Logistic regression is linear; complex nonlinear interactions may not be fully captured.

**Potential Next Steps:**

1. **Test more advanced models**  
   - Random Forest, Gradient Boosting (XGBoost/LightGBM) for potentially higher ROC-AUC.
2. **Calibration and thresholds**  
   - Tune the probability threshold for defining “high risk” based on the cost of churn vs. retention offers.
3. **Uplift / treatment modeling**  
   - Focus not just on who will churn, but on who is **most likely to stay if contacted**.
4. **Operationalization**
   - Integrate scores into CRM tools so sales/retention teams can see **churn risk** next to each customer.
   - Schedule monthly score refreshes as new data arrives.

---

## 9. Conclusion

This project demonstrates how a **simple, interpretable model** plus a **clear dashboard** can:

- Identify **which customers are likely to churn**,
- Reveal **why** they churn (contract type, tenure, charges, payment method),
- And support **practical, targeted retention strategies**.

Used correctly, these insights can help the business **reduce churn, protect recurring revenue, and focus retention efforts where they have the highest impact**.