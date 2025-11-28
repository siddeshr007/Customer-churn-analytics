## Customer Churn Analytics (Telecom)

This project analyzes telecom customer churn using:

    - **Python / scikit-learn** for data prep and machine learning
    - **Tableau Public** for an interactive churn risk dashboard
    - **BA-style documentation** for insights and business recommendations

The goal is to predict which customers are likely to churn and translate those predictions into clear, actionable steps for a retention team.

---

## 🎯 Business Question

* A telecom provider wants to reduce customer churn by answering:
    - Which customers are most likely to churn?
    - What are the key drivers of churn (contract, tenure, charges, payment method, etc.)?
    - How can the business prioritize retention efforts based on predicted churn risk?

---

## 📂 Project Structure

~~~
customer-churn-analytics/
├── data/
│   ├── raw/
│   │   └── telco_churn_raw.csv          # Original telco churn dataset
│   └── processed/
│       └── churn_scored.csv             # Test set with model churn probabilities
├── docs/
│   ├── churn_report.md                  # Business / BA-style narrative report
│   └── churn_dashboard.png              # Exported Tableau dashboard preview
├── models/
│   └── churn_logistic_model.pkl         # Saved logistic regression model
├── notebooks/
│   └── 01_churn_model.ipynb             # Main Jupyter notebook (EDA + ML)
├── .gitignore
└── README.md
~~~

---

## 📊 Data

* Source: Public Telco Customer Churn dataset (e.g., Kaggle)
* Unit of analysis: 1 row = 1 customer
Main feature groups
    * Customer info: customerID, gender, SeniorCitizen, Dependents
    * Services: phone, internet, streaming, tech support, online backup, etc.
    * Account: Contract, PaymentMethod, PaperlessBilling, tenure, MonthlyCharges, TotalCharges
    * Target: Churn (Yes / No)
After cleaning
    * Total customers: ~7,043
    * Churn rate: ~26–27% of customers

## 🧮 Modeling (notebooks/01_churn_model.ipynb)
* Preprocessing
In the notebook, the data is loaded and prepared:
```
import pandas as pd
df = pd.read_csv("data/raw/telco_churn_raw.csv")
```
Key preprocessing steps:
- Standardized column names (lowercase, underscores).
- Converted TotalCharges to numeric and dropped rows with invalid / missing values.
- Encoded Churn as:
    - Yes → 1 (churner)
    - No → 0 (non-churner)
- One-hot encoded categorical features using pd.get_dummies(drop_first=True).

Train–test split:
```
from sklearn.model_selection import train_test_split

X = df.drop(["churn", "customerid"], axis=1, errors="ignore")
y = df["churn"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y
)
```
* Model
- Algorithm
    - Logistic Regression (sklearn.linear_model.LogisticRegression)
    - Parameters: max_iter = 1000
- Evaluation on the test set
    - Accuracy: ~80%
    - Class 1 (Churn) metrics:
        - Precision ≈ 65%
        - Recall ≈ 57%
        - F1-score ≈ 0.61
    - ROC-AUC: ~0.84
- Interpretation
    - The model is reasonably accurate overall (8 of 10 predictions correct).
    - Precision ~65% → when the model flags a customer as high-risk, it is right about two-thirds of the time.
    -Recall ~57% → the model captures over half of all churners, a solid starting point for targeted campaigns.
    - ROC-AUC ~0.84 → strong separation between churners and non-churners, better than simple rule-based heuristics.

The trained model is saved for reuse:
```
import joblib

joblib.dump(model, "models/churn_logistic_model.pkl")
```
---

## 🔑 Key Drivers of Churn
Using the logistic regression coefficients and follow-up analysis, several features stand out:
* Tenure
    - Customers with shorter tenure have a much higher churn probability.
    - Long-tenure customers are more stable and less likely to leave.
* Contract Type
    - Month-to-month contracts show substantially higher churn rates.
    - One-year and two-year contracts are associated with much lower churn.
* Charges
    - Higher MonthlyCharges are correlated with higher churn risk, suggesting price sensitivity.
* Payment Method
    - Certain payment methods (e.g., Electronic check) show higher churn compared to more “sticky” methods such as bank transfer or credit card.
These drivers are visualized in the Tableau dashboard to make them easy to explain to non-technical stakeholders.

---

## 📁 Scored Dataset
To feed Tableau, the model’s predictions on the test set are exported:
```
X_test_copy = X_test.copy()
X_test_copy["churn_actual"] = y_test.values
X_test_copy["churn_probability"] = y_prob  # model.predict_proba(X_test)[:, 1]

X_test_copy.to_csv("data/processed/churn_scored.csv", index=False)
```

"data/processed/churn_scored.csv" includes:
    - churn_actual (0/1) – true churn flag
    - churn_probability (0–1) – predicted probability of churn
    - All original feature columns for each test customer
This file is the data source for the Tableau dashboard.

---

## 📈 Tableau Dashboard

Built on data/processed/churn_scored.csv, the Tableau dashboard includes:
- Churn Probability Distribution
    - Histogram of predicted churn probabilities.
    - Shows most customers are low-risk, with a smaller tail of high-risk customers.
- Average Churn Probability by Contract Type
    - Month-to-month vs 1-year vs 2-year contracts.
    - Month-to-month customers display the highest churn risk.
- Average Churn Probability by Tenure Band
    - Bands such as < 1 year, 1–2 years, 2–4 years, 4–6 years, 6+ years.
    - Short-tenure customers are much more likely to churn.
- Average Churn Probability by Payment Method
    - Electronic check vs credit card vs bank transfer vs mailed check.
    - Highlights riskier payment channels, especially electronic check.
The dashboard is exported as an image for quick preview.

---

## 🖼️ Dashboard Preview

Below is a preview of the Tableau churn dashboard built from the scored dataset:

For full interactivity, open the Tableau workbook and connect it to data/processed/churn_scored.csv.

---

## 💡 Business Takeaways

* From the model and dashboard:
- Churn is concentrated among:
    - Short-tenure customers
    - On month-to-month contracts
    - With higher monthly charges
    - Using certain payment methods (especially electronic check)
* Potential actions:
- Protect month-to-month and new customers
    - Offer welcome journeys, early-tenure check-ins, and targeted incentives to move to longer-term contracts.
- Review pricing for high-charge customers
    - Consider bundles, discounts, or loyalty benefits for customers paying at the high end of monthly charges.
- Encourage stickier payment methods
    - Promote auto-pay via bank transfer or credit card (with small incentives) to reduce churn among electronic-check users.
- Use churn probability for prioritization
    - Focus retention outreach on the top-risk deciles rather than treating all customers the same.
A narrative version of these findings is documented in:

## 📄 docs/churn_report.md

---

## 🛠️ Tools & Skills Used

* Python / Jupyter
    - pandas, numpy, scikit-learn, joblib
    - Data cleaning, feature engineering, model training & evaluation
* Machine Learning
    - Logistic Regression
    - Binary classification metrics (accuracy, precision, recall, F1, ROC-AUC)
* Tableau Public
    - Visual exploration of churn probabilities and key drivers
    - Interactive charts for contract type, tenure, and payment method
* Git & GitHub
    - Version control for notebooks, models, and documentation
* Business Analysis
    - Framing the business problem
    - Translating model outputs into actionable recommendations

🚀 How to Run Locally

* Clone the repo
```
git clone https://github.com/<your-username>/customer-churn-analytics.git
cd customer-churn-analytics
```

* Create & activate a virtual environment (optional but recommended)
```
# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python -m venv .venv
source .venv/bin/activate
```
* Install dependencies
```
pip install -r requirements.txt  # if present
```
* Run the notebook
```
jupyter notebook notebooks/01_churn_model.ipynb
```
* Open the Tableau dashboard   
    - In Tableau Public / Desktop, connect to:
        data/processed/churn_scored.csv
    - Recreate or refresh the dashboard using the fields described above.

---

##✍️ Author

Sai Siddesh Reddy Bynigeri
Business / Data Analyst – Python, SQL, Tableau, Excel
