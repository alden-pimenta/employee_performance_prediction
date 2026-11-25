# INX Future Inc. – Employee Performance Analysis
**IABAC Certified Data Scientist Project – Code 10281**

## 1. Project overview
- **Algorithm used:** Gradient Boosting classifier (selected from 7 algorithms plus a baseline), inside a scikit-learn pipeline with one-hot encoding and scaling.
- **Important features:** environment satisfaction, last salary hike %, years since last promotion.
- **Tools and techniques:** Python, pandas, NumPy, Matplotlib, seaborn, SciPy, scikit-learn, Jupyter. Chi-square tests, Spearman correlation, stratified cross-validation, grid search, permutation importance.
- **Headline result:** accuracy 92.9% and macro-F1 0.89 on 240 unseen employees (baseline "always 3": 72.9% and 0.28).

## 2. Business problem
INX Future Inc. is a well-regarded employer, but employee performance has declined, escalations have risen and client satisfaction fell by 8 percentage points. The CEO wants the root causes, clear indicators of low performers (so any action does not damage the morale of everyone else), a predictive model, and recommendations.

## 3. Dataset description
1200 employees, 28 columns (employee ID, demographics, department and role, satisfaction scores, pay and experience variables, overtime, attrition) and the target `PerformanceRating` (2 = 16.2%, 3 = 72.8%, 4 = 11.0%). The file is stored unchanged in `data/raw`; column definitions are in `references/data_definitions.csv`.

## 4. Data preprocessing methodology
No missing values, duplicate rows or duplicate IDs; no impossible values (checked: experience ordering, age, score ranges). Outliers (IQR rule) were real values and kept. `EmpNumber` was dropped. Cleaned data: `data/processed/employee_clean.csv`. Encoding and scaling are done inside the modelling pipeline so that nothing leaks from the test set.

## 5. Exploratory data analysis
Univariate, bivariate and multivariate analysis is in `src/Data Processing/data_exploratory_analysis.ipynb`. Main findings:
- Ratings are concentrated at 3.
- Environment satisfaction is the strongest link: **40% of employees at satisfaction 1-2 are rated 2, against 1% at satisfaction 3-4.** This holds in all six departments.
- Salary hike: **61% of employees whose last hike was 20% or more are rated 4, against 2% for smaller hikes.**
- Promotion: employees promoted within the last year average 3.12; those waiting 2-3 years average about 2.77 (longer waits have few employees and noisy averages).
- No meaningful link with rating: number of training sessions last year, job satisfaction, relationship satisfaction, job involvement, gender, marital status, education background, travel frequency, attrition. Overtime is statistically significant but weak and in the opposite direction to expectation.

## 6. Feature selection and engineering
Correlation analysis, chi-square tests, and permutation importance were used. Categorical columns were one-hot encoded, numeric columns standardised, inside the pipeline. Highly correlated experience columns were kept because tree models tolerate them and importance is measured per original column. A model using only the top 5 columns performs identically to the full model.

## 7. Machine learning algorithms and evaluation
Logistic Regression, KNN, SVM, Decision Tree, Random Forest, Gradient Boosting and a neural network (MLP) were compared with 5-fold CV and an unseen test set, using accuracy and macro-F1 (see `Project Summary/Analysis/Analysis.md` for the full table). The tree-based models are clearly ahead of the others. Gradient Boosting was selected because it had the best tuned CV macro-F1 (0.893); it reaches accuracy 0.929 and macro-F1 0.891 on the test set. Recall for rating 4 is the weakest point (0.77).

## 8. Important findings

### 8.1 Insight 1 – Department-wise performance
| Department | Employees | Avg rating | % rated 2 | % rated 4 | Employees rated 2 |
|---|---|---|---|---|---|
| Development | 361 | 3.09 | 3.6% | 12.2% | 13 |
| Data Science | 20 | 3.05 | 5.0% | 10.0% | 1 |
| Human Resources | 54 | 2.93 | 18.5% | 11.1% | 10 |
| Research & Development | 343 | 2.92 | 19.8% | 12.0% | 68 |
| Sales | 373 | 2.86 | 23.3% | 9.4% | 87 |
| Finance | 49 | 2.78 | 30.6% | 8.2% | 15 |

The differences are statistically significant (chi-square p < 0.001). Sales and R&D hold 155 of the 194 employees rated 2, so they account for most of the problem in absolute numbers; Finance has the highest rate. Lowest-rated job roles: Finance Manager (2.78), Senior Manager R&D (2.80), Healthcare Representative (2.85), Laboratory Technician, Sales Executive and Sales Representative (about 2.86-2.87). Development employees with low environment satisfaction still average about 3.0, whereas in Finance they average about 2.2, so Finance, R&D and Sales are far more sensitive to a poor environment.

### 8.2 Insight 2 – Top 3 influencing factors
1. **Employee environment satisfaction** (permutation importance 0.294). Low scores go with most low ratings: 97% of employees rated 2 have a score of 1-2, against 28% of everyone else.
2. **Last salary hike %** (0.269). Large hikes (20%+) are strongly associated with rating 4.
3. **Years since last promotion** (0.100). Employees rated 2 average 3.7 years since promotion, others 1.9.

These are strong associations, not proof of cause: a good rating may lead to a bigger raise, and a poor rating may lower satisfaction. They should be tested through pilots (Section 10).

### 8.3 Insight 3 – Prediction models
- **Current-employee model:** Gradient Boosting, accuracy 0.929, macro-F1 0.891 (test set). It takes raw employee records and handles encoding and scaling inside the pipeline.
- **Hiring model (candidate information only):** accuracy 0.517, macro-F1 0.380, **below** the 0.729 baseline. Candidate background (age, education, role, experience and so on) does not reliably predict later rating in this data. **The model must not be used to accept or reject candidates.**
- **Identifying low performers (CEO's request):** `reports/employee_risk_scores.csv` ranks every current employee by the model's probability of rating 2. Employees at high risk typically combine environment satisfaction 1-2, a small raise and a long gap since promotion. Use it as a support and review list, because these scores are on data the model has already seen.

## 9. Does the solution address the business problem?
Partly, and honestly: it identifies *where* performance is weak (departments, roles), *what* it is most strongly associated with (environment, pay growth, promotion), and gives a reliable model for current employees. It does **not** provide a trustworthy hiring predictor, and it cannot by itself prove the causes. The data has no information on managers, workload, client feedback or time trends, which are likely to matter.

## 10. Business recommendations
Each recommendation is tied to a finding above. Items marked *(assumption to test)* go beyond what the data can show.

**Department-wise**
- *Sales and R&D (155 of 194 low ratings):* start environment reviews here first because of the sheer number of employees affected; include Sales Executives (270 people, avg 2.86) and Laboratory Technicians.
- *Finance (30.6% rated 2, highest rate; gap between low and high satisfaction is the widest):* run focused listening sessions and review role expectations for Finance Managers (avg 2.78). Small team (49), so a quick pilot is feasible.
- *HR:* 18.5% rated 2; monitor, with a lighter-touch review.
- *Development and Data Science:* document what works (team set-up, manager practices) and share it. *(assumption to test: these practices are transferable)*

**Improving employee satisfaction (the strongest factor)**
- Run an environment satisfaction survey every six months and follow up with every employee scoring 1-2 (about 39% of employees; 40% of them are rated 2).
- Ask what is behind low scores (workspace, team, tools, workload) before choosing fixes. The data does not say which element matters. *(assumption to test)*
- Track whether ratings of employees whose satisfaction improves also change, to learn whether the link is causal.

**Training and development**
- The number of training sessions last year shows **no relationship** with rating (correlation 0.00), so adding more generic training is not supported by the data. Prefer targeted, role-specific development for those with low ratings, and measure results.
- Review promotion timing: employees waiting 2+ years since promotion average lower ratings. Create development paths or lateral moves for people waiting longest.

**Strategies to improve performance**
- Review how salary hikes are decided. 84% of rating-4 employees received a hike of 20% or more, and 61% of such hikes went to rating-4 employees; make sure raises are transparent and that rating-3 employees (73% of staff) can see a path to recognition.
- **Handle low performers with support first**: use the risk list for coaching conversations and root-cause discussions, not for penalties. Because the strongest drivers are conditions INX controls, blanket penalties would hit many people for a workplace problem and risk the morale damage the CEO fears.
- Re-measure each half-year: rating mix by department, environment satisfaction, and time since promotion.

**Recruitment**
- **Do not use the hiring model to decide on candidates.** It is worse than guessing "everyone will be a 3".
- Use structured interviews and role-specific skill assessments, and start recording pre-hire assessment scores so that a future hiring model can be built on relevant data.
- Because performance appears tied to the environment after joining, invest in onboarding and early satisfaction checks for new hires.
- Review for fairness: gender, marital status and education background show no link with rating and must not be used in hiring decisions.

## 11. Final conclusion
Performance at INX is concentrated in a clear pattern: low ratings sit mostly in Sales, R&D and Finance and are strongly associated with low environment satisfaction, small salary hikes and long gaps since promotion. A boosted-tree model identifies these cases on unseen employees with about 93% accuracy (macro-F1 0.89), which gives management an objective support list. Candidate background does not predict performance, so hiring should not rely on this model. The most actionable path is to improve the work environment and promotion and pay transparency, starting in Sales, R&D and Finance, and to verify the effect through a pilot with before/after measurement.