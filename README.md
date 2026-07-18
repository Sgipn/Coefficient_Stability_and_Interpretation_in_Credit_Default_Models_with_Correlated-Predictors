# Feature Cost vs. Signal in Credit Risk Prediction

### Overview
This project examines how much of a credit default model's predictive signal can be retained when the number and cost of input features is deliberately constrained. Financial variables (e.g., credit limits, balances) and behavioral variables (e.g., repayment history) are not independent signals; they encode overlapping information about borrower risk, and acquiring and maintaining each additional feature carries a real data-acquisition cost.

The analysis extends beyond predictive accuracy alone to ask which features are worth their cost, how that answer depends on the borrower's current state, and how a small, low-cost feature set compares against the full specification.

### Questions
- Do financial variables retain explanatory power after controlling for repayment behavior?
- Can repayment history be represented using lower-dimensional summary features without losing signal?
- Which cost drivers justify their data-acquisition cost, and which can be cut without sacrificing predictive power?
- Does variable importance shift depending on the borrower's delinquency regime, and is that shift statistically significant?
- How do linear specifications compare against a nonlinear benchmark once the feature set is reduced?

### Data
- Source: UCI Default of Credit Card Clients
- Observations: 30,000
- Target: Default in the following month (binary)

##### Feature groups:
- Financial variables: credit limit, bill amounts, payment amounts
- Behavioral variables: repayment status over time (PAY_0–PAY_6)
- Demographic variables (marriage, sex, age)
- The dataset is drawn from 2005. While absolute levels of default risk may differ in current settings, the analysis focuses on structural relationships between variables and on the cost/signal tradeoff, not on producing a production-ready risk score.

### Approach

##### Exploratory Analysis
- A strong monotonic relationship between delinquency and default
- High correlation within financial variables and within repayment variables
- Evidence of redundancy across predictors

##### Feature Construction
Repayment behavior is summarized into:
- ANY_DELAY
- MAX_DELAY
- NUM_DELAY_MONTHS

These features capture the presence, severity, and persistence of delinquency while reducing dimensionality and, by extension, the number of fields that would need to be collected and maintained in production.

##### Model Specifications
11 model specifications were built and stress-tested across covariate sets, including:
- Financial variables only
- Behavioral variables only (full repayment history)
- Behavioral variables only (summary features)
- Financial + behavioral variables
- Financial + summary behavioral features
- Reduced 2-variable specification

##### Regularization
LASSO and Ridge regularization paths were used to trace how coefficients shrink and which variables survive selection as the penalty increases, extending the specification comparison into a continuous view of the cost/signal tradeoff rather than a fixed set of discrete models.

##### Evaluation
- Metric: out-of-sample AUC, plus asymmetric threshold optimization to weigh the cost of a missed default against the cost of a false flag, rather than treating all classification errors as equivalent.
- Method: train/test split
- Focus: comparison across specifications and cost tradeoffs, not accuracy maximization alone

##### Regime Dependence
A likelihood ratio test (regime LRT) was used to formally test whether variable importance differs between non-delinquent and delinquent borrower states, rather than relying on a descriptive comparison alone.

##### Nonlinear Benchmark
An XGBoost model was fit on the reduced feature set to check whether a flexible, nonlinear model could extract meaningfully more signal than the linear specifications once the feature set was cut down.

##### Interpretation
Regression is treated as a projection rather than a structural model: coefficients represent conditional contributions, and a variable's coefficient reflects what it explains beyond what is already captured by other predictors. This distinction is central both to interpreting correlated covariates and to deciding which features are worth their cost.

### Results

##### Predictive Performance
Behavioral variables substantially outperform financial variables in isolation. Combined models yield only modest improvements, indicating that much of the predictive content of financial variables overlaps with repayment behavior.

##### Feature Representation
Summary measures of delinquency perform comparably to the full repayment history, and a 2-variable model matched full-model accuracy. This suggests the behavioral signal is effectively low-dimensional and that most of the feature set can be cut without a meaningful accuracy cost.

##### Coefficient Stability
Financial variables appear important in isolation but weaken when behavioral variables are included. This reflects multicollinearity and the conditional nature of coefficient interpretation.

##### Regime Dependence
Variable importance depends on borrower state, and the regime LRT confirmed this difference is statistically significant:
- In non-delinquent regimes, financial variables provide more information
- In delinquent regimes, behavioral variables dominate

##### Nonlinear Check
The XGBoost benchmark did not meaningfully outperform the reduced linear specifications, reinforcing that the signal in this dataset is concentrated in a small number of behavioral features rather than in complex nonlinear interactions.

### Takeaways
- Default risk is primarily driven by realized delinquency
- Financial variables act as indirect or leading indicators of risk
- A small, low-cost feature set retains nearly all of the predictive signal, supporting a trade-off recommendation to cut scope without sacrificing signal
- Coefficient estimates are sensitive to specification under correlated predictors, and interpretation is more meaningful at the level of variable groups than individual coefficients
- Which features are worth keeping depends on borrower regime, not just aggregate performance

This project emphasizes the distinction between predictive performance and interpretation, and treats feature cost as a first-class part of the model comparison rather than an afterthought. In settings with correlated, costly-to-acquire predictors, understanding how information and cost are distributed across variables is often more important than marginal improvements in predictive accuracy.
