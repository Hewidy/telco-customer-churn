# Customer Churn Prediction

## Business problem

A telecom company wants to identify customers who are likely to leave so it can offer support early. This project uses customer information to predict `Churn`: Yes means the customer left, and No means the customer stayed. This is classification because the prediction is a category. Churn = Yes is the positive class. Missing a customer who will leave can mean losing future income and a chance to resolve their problem.

## Data and main findings

The IBM Telco Customer Churn dataset contains 7,043 customers and 21 columns. There were no duplicate rows. `TotalCharges` contained 11 blank strings, which became missing values after conversion to numbers.

The exploration produced four main findings:

1. **Churn was the smaller group:** 26.54% of customers left, while 73.46% stayed. Accuracy alone can therefore be misleading.
2. **Contract type was associated with churn:** churn rates were 42.71% for month-to-month contracts, 11.27% for one-year contracts, and 2.83% for two-year contracts.
3. **Customers who left had shorter tenure:** their median time with the company was 10 months, compared with 38 months for customers who stayed.
4. **Customers who left had higher monthly charges:** their median charge was 79.65, compared with 64.43 for customers who stayed.

These patterns show associations. They do not prove that contract type, tenure, or charges caused customers to leave.

## Preparation and model comparison

The identifier `customerID` was removed. The data was split into 5,634 training customers and 1,409 test customers, using `random_state=42` and stratification to preserve the churn proportions. Numerical missing values were filled with the training median, and numerical inputs were scaled. Categorical missing values were filled with the most common training value, and categories were encoded as numerical indicators.

Preprocessing was included inside each model pipeline. During five-fold cross-validation, preprocessing was fitted only on each training fold. Model selection and tuning used the training data; the held-out test set was reserved for final evaluation.

The table shows mean cross-validation scores on the training data. Precision, recall, and F1 refer to customers who churned.

| Model | Accuracy | Precision | Recall | F1 |
| --- | ---: | ---: | ---: | ---: |
| Baseline: predict everyone stays | 0.735 | 0.000 | 0.000 | 0.000 |
| Logistic Regression | 0.803 | 0.655 | 0.543 | 0.593 |
| Decision Tree | 0.730 | 0.491 | 0.498 | 0.494 |
| Random Forest | 0.784 | 0.624 | 0.475 | 0.539 |
| Tuned Logistic Regression | 0.750 | 0.519 | 0.797 | 0.629 |

Recall is the main business priority because it measures how many actual churners are detected. Precision also matters because false alerts can waste staff time or offers. F1 balances precision and recall and was used to select the tuning settings.

## Tuning and final results

Logistic Regression had the highest initial cross-validation F1 and recall. GridSearchCV tested four values of `C` and two class-weight settings. The selected settings were `C=0.1` and `class_weight="balanced"`.

Tuning improved cross-validation F1 from 0.593 to 0.629, a modest increase of 3.6 percentage points. Recall increased substantially, from 0.543 to 0.797, but precision and accuracy decreased. This trade-off means more churners were identified at the cost of more false alerts.

The chosen model was then evaluated on the held-out test set. It achieved **74.2% accuracy, 50.9% precision, 78.6% recall, and 61.8% F1**.

| Actual outcome | Predicted stay | Predicted churn |
| --- | ---: | ---: |
| Stayed | 751 true negatives | 284 false positives |
| Churned | 80 false negatives | 294 true positives |

The model detected 294 of the 374 customers who left, but missed 80. It also flagged 284 customers who actually stayed. About half of its churn alerts were correct.

## Recommendation and limitations

Use the tuned Logistic Regression in a small retention pilot. Its higher recall fits the goal of identifying customers who may leave early. First, contact flagged customers to understand and resolve service problems. Second, review their needs and offer suitable plans or targeted retention offers when appropriate.

The classes are imbalanced, and false alerts could make expensive offers wasteful. This teaching dataset may also differ from current customers. Finally, predicting churn does not show that an intervention will prevent it. Start with low-cost support calls, then measure retention outcomes and intervention costs before expanding the approach.
