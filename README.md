# Team Name: DSM

## Contributors

- Delaney — Linear Regression
- Maitreyee — Logistic Regression
- Snehal — Generalized Additive Model (GAM)

## Dataset


Used the Telco Customer Churn dataset to predict whether a telecom
customer will leave the company.

The original dataset contains 7,043 customer records. Each row represents
one customer and includes demographic characteristics, subscribed
services, contract details, payment information, and billing information.
`Churn` is the target variable, with `Yes` indicating that the customer
churned and `No` indicating that the customer stayed.

For data cleaning, `customerID` was removed because it is an identifier
rather than a meaningful predictor. `TotalCharges` was converted from
text to a numeric variable. Eleven rows with missing `TotalCharges`
values were removed, leaving 7,032 observations for analysis. Churn was
then encoded as a binary outcome, where 0 represents staying and 1
represents churning.

The cleaned dataset has a moderately imbalanced target. Approximately
73.4% of customers stayed, while 26.6% churned. Because non-churners are
the majority class, accuracy alone may give an incomplete picture of
model performance. We therefore also considered measures such as
precision, recall, F1 score, and ROC-AUC.

The predictors include several types of variables:
- Continuous numerical variables: `tenure`, `MonthlyCharges`, and
`TotalCharges`
- A binary variable stored numerically: `SeniorCitizen`
- Binary categorical variables describing characteristics such as
partner status, dependents, phone service, and paperless billing
- Multi-category variables such as `Contract`, `InternetService`, and
`PaymentMethod`

Categorical predictors were encoded so that they could be used by the
models. The EDA also showed that some predictors contain overlapping
information. In particular, `tenure` and `TotalCharges` were strongly
correlated, with a correlation of approximately 0.826. Several
internet-service variables were also structurally related. This
predictor redundancy creates concerns when interpreting individual
coefficients or GAM smooth effects.

The EDA suggested that some continuous predictors do not have simple
straight-line relationships with churn. Tenure showed especially clear
curvature. This motivated checking the linearity-in-the-log-odds
assumption for logistic regression and comparing logistic regression
with a GAM that allows nonlinear smooth effects.

Each model used the same stratified 80/20 train-test split, with a fixed
random seed. This produced 5,625 training observations and 1,407
held-out test observations. Stratification kept the churn proportions
approximately equal in the training and test sets. No class-resampling
method was applied, so the models were trained using the churn
distribution observed in the cleaned dataset.


## Assumption Checks

| Model | Key Assumptions Checked | Evidence | Concern |
|---|---|---|---|
| Linear regression | Linearity, independence, homoscedasticity, normal residuals, multicollinearity, influential observations | RESET, Durbin-Watson, Breusch-Pagan, Jarque-Bera, VIF, and Cook's Distance | Linearity, constant variance, normality, and no-multicollinearity assumptions were not confirmed. |
| Logistic regression | Binary outcome, independence, separation, linearity in log-odds, multicollinearity, sample size | Churn encoding, duplicate-ID check, cross-tabs, Box-Tidwell, VIF, and events-per-variable | Evidence of nonlinearity and elevated VIF values creates concerns for coefficient interpretation. |
| GAM | Appropriate outcome, nonlinear effects, additive structure, independence, predictor redundancy, data quality | Logistic GAM, EDA, smooth-effect plots, duplicate-ID check, and correlation checks | Additive model may miss interactions; `tenure` and `TotalCharges` are strongly correlated (≈0.826). |

## Model Comparison

| Model | Performance Evidence | Interpretability Strength | Interpretability Weakness |
|---|---|---|---|
| Linear regression | Test accuracy = 79.5% at a 0.50 threshold; R² = 0.260; MSE = 0.145 | Simple, direct coefficients that show the direction and magnitude of associations | Not designed for a binary outcome; 15.6% of predictions fell outside 0–1; classification depends on a selected threshold; several assumptions were not met. |
| Logistic regression | Test accuracy = 80.5%; ROC-AUC = 0.836; F1 = 0.611 | Appropriate for binary churn and coefficients can be expressed as odds ratios | Continuous predictors are assumed to be linear in the log-odds; Box-Tidwell and VIF results raise concerns. |
| GAM | Test accuracy = 79.6%; ROC-AUC = 0.841 | Captures nonlinear relationships while remaining interpretable through smooth-effect plots | Additive structure does not capture interactions automatically, and correlated predictors can make individual effects difficult to interpret. |


## Recommendation

**Recommended model:**  
Logistic Regression

**Why this model:**  
Logistic regression provides the best balance of simplicity, interpretability, and predictive performance for this churn task. It is designed for a binary outcome, its coefficients can be explained using odds ratios, and its test accuracy was 80.5%, slightly higher than the GAM's 79.6%. Its ROC-AUC of 0.836 was also very close to the GAM's 0.841. Although the GAM can capture nonlinear relationships, its small AUC advantage does not clearly outweigh the simpler interpretation of logistic regression. Linear regression was not selected because it is less appropriate for a binary outcome and several of its assumptions were not supported.

**What the company can responsibly conclude:**  
The logistic-regression model demonstrated useful ability to distinguish customers who churned from customers who stayed, with a held-out ROC-AUC of 0.836. The model can be used to  estimate and rank customer churn risk.

The fitted coefficients identify customer characteristics that are conditionally associated with higher or lower estimated churn odds. These associations can help the company investigate risk patterns and prioritize customers for retention review.

The analysis also suggests that some continuous predictors, particularly tenure, may have nonlinear relationships with churn. Therefore, a future version of the logistic model could include a targeted spline or transformation for tenure while retaining simpler coefficient-based effects for the other predictors.

**What the company should not conclude yet:**  
The model shows associations, not causation. The company should also be cautious when interpreting individual coefficients because the assumption checks found concerns with linearity in the log-odds and multicollinearity.

**One next analysis we would run:**  
Test nonlinear terms or interactions for important continuous predictors and compare the resulting model with the current logistic regression to determine whether its linearity assumption is limiting performance.
