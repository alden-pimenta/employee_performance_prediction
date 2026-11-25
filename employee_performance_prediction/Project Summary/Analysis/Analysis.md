# Analysis, Algorithm Selection and Decisions

## 1. Problem framing
- **Target:** `PerformanceRating` with values 2, 3, and 4 (ordered, 3 classes) → **classification**.
- **Class balance:** rating 2 = 194 (16.2%), rating 3 = 874 (72.8%), rating 4 = 132 (11.0%). Because of this imbalance, accuracy alone is misleading: always predicting "3" scores 72.9% accuracy but a macro-F1 of only 0.281. **Macro-F1** is the main selection metric.

## 2. Data preparation decisions

| Decision | Reason |
|---|---|
| No rows removed, nothing imputed | 0 missing values, 0 duplicate rows or IDs, no impossible values |
| Outliers kept | Real values (long careers); tree models are not sensitive |
| `EmpNumber` dropped | Identifier, no information |
| One-hot encoding for categorical columns | No natural order between categories |
| Standard scaling for numeric columns | Needed by Logistic Regression, KNN, SVM and MLP; harmless for trees |
| Encoding and scaling inside a `Pipeline` | Fitted on training data only (no leakage); same transformation at prediction time |
| Stratified 80/20 split (960 train / 240 test), 5-fold stratified CV | Keeps class proportions |
| Correlated columns kept | Age, experience and tenure columns are highly correlated, but tree ensembles tolerate this; importance is measured on whole original columns |

## 3. Algorithms compared

Cross-validation on the training set (5 folds) and one evaluation on the unseen test set. Models were compared primarily using macro-F1 to account for class imbalance.

| Model | CV accuracy | CV macro-F1 | Test accuracy | Test macro-F1 |
|---|---|---|---|---|
| Gradient Boosting | 0.929 | 0.889 | 0.929 | 0.886 |
| Decision Tree | 0.888 | 0.837 | 0.908 | 0.869 |
| Random Forest | 0.900 | 0.822 | 0.912 | 0.855 |
| SVM (RBF) | 0.770 | 0.700 | 0.767 | 0.689 |
| Neural Network (MLP) | 0.801 | 0.680 | 0.825 | 0.703 |
| Logistic Regression | 0.736 | 0.668 | 0.758 | 0.687 |
| KNN | 0.739 | 0.514 | 0.729 | 0.527 |
| Baseline (always rating 3) | 0.728 | 0.281 | 0.729 | 0.281 |

The results indicate that **Gradient Boosting performed best overall**, achieving the highest CV macro-F1 (0.889) and test macro-F1 (0.886), while also maintaining high accuracy of 0.929 on both validation and test evaluation.

Random Forest and Decision Tree also performed relatively well, but their macro-F1 scores were lower than Gradient Boosting. SVM, MLP, Logistic Regression, and KNN showed comparatively weaker performance.

## 4. Model selection

**Gradient Boosting was selected as the best-performing model based on the reported cross-validation macro-F1 of 0.889.**

It also achieved a test accuracy of 0.929 and test macro-F1 of 0.886, demonstrating consistent performance between cross-validation and the unseen test set.

The consistency between CV and test results suggests that the model generalizes reasonably well to unseen observations, with no obvious indication of substantial overfitting based on these metrics.

The tree-based ensemble models performed better than the other tested approaches, indicating that the relationships between employee attributes and performance ratings may involve non-linear patterns and interactions that these models can capture effectively.

## 5. Performance stability

The cross-validation standard deviations reported for the models provide an indication of performance variability across folds.

| Model | CV accuracy std | CV macro-F1 std |
|---|---|---|
| Gradient Boosting | 0.005 | 0.010 |
| Decision Tree | 0.019 | 0.023 |
| Random Forest | 0.008 | 0.024 |
| SVM (RBF) | 0.022 | 0.025 |
| Neural Network (MLP) | 0.021 | 0.036 |
| Logistic Regression | 0.019 | 0.023 |
| KNN | 0.026 | 0.057 |
| Baseline (always rating 3) | 0.002 | 0.000 |

Gradient Boosting showed relatively low variation across folds, with a CV accuracy standard deviation of 0.005 and macro-F1 standard deviation of 0.010. This indicates relatively consistent performance across the cross-validation folds.

## 6. Additional test metrics

The screenshot also reports test macro-precision and macro-recall:

| Model | Test macro-precision | Test macro-recall |
|---|---|---|
| Gradient Boosting | 0.913 | 0.862 |
| Decision Tree | 0.837 | 0.910 |
| Random Forest | 0.938 | 0.802 |
| SVM (RBF) | 0.663 | 0.749 |
| Neural Network (MLP) | 0.748 | 0.672 |
| Logistic Regression | 0.652 | 0.758 |
| KNN | 0.621 | 0.494 |
| Baseline (always rating 3) | 0.243 | 0.333 |

Gradient Boosting offers a strong balance between macro-precision and macro-recall. Random Forest achieved the highest test macro-precision (0.938), while Decision Tree achieved the highest test macro-recall (0.910) among the non-baseline models. However, Gradient Boosting remains the strongest overall choice based on the primary selection metric, macro-F1.

## 7. Limitations
- One snapshot of 1200 employees; no manager, client or time-series data.
- Ratings may themselves contain bias.
- Associations are not causes: employee satisfaction and salary changes may be related to performance ratings without directly causing them.
- Model performance on the test set represents performance on this particular split and may vary on new data.
- The model should support analytical insights rather than be used as the sole basis for employee-related decisions.
