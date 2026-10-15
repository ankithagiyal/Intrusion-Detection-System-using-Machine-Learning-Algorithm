# Intrusion Detection System using Machine Learning

A machine learning pipeline that detects network intrusions in the NSL-KDD
dataset using decision trees. It classifies traffic as normal or as one of
four attack categories: DoS, Probe, R2L and U2R.

## Overview
Intrusion detection systems flag malicious network traffic. This project
builds a detector for each major attack category, compares feature selection
methods, and evaluates the models with confusion matrices and
cross-validation.

## Dataset
[NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html) (Tavallaee et al., 2009), an improved version of the KDD Cup 99 dataset.
- **Training set:** `KDDTrain+_2.csv` (125,973 records)
- **Test set:** `KDDTest+_2.csv` (22,544 records)
- 41 features per connection (protocol, service, flag, byte counts, error rates, host statistics)
- Attack labels grouped into four categories:

| Category | Meaning |
|----------|---------|
| DoS | Denial of service: exhausts the victim's resources |
| Probe | Surveillance and scanning to gather information |
| R2L | Remote to local: unauthorised access from a remote machine |
| U2R | User to root: escalating to administrator privileges |

## Approach
1. **Preprocessing:** label-encode and one-hot-encode `protocol_type`,
   `service` and `flag` (84 dummy columns); map the 38 attack labels to
   four categories
2. **Dataset split by attack type:** four separate normal-vs-attack datasets
   (DoS, Probe, R2L, U2R)
3. **Feature scaling:** `StandardScaler`
4. **Feature selection:**
   - Univariate selection with the ANOVA F-test (top 10% of features)
   - Recursive Feature Elimination (RFE) with a decision tree, keeping the
     best 13 features per attack type
5. **Model:** `DecisionTreeClassifier`, one per attack category
6. **Evaluation:** confusion matrices and 10-fold cross-validation
   (accuracy, precision, recall, F-measure)

## Results

**10-fold cross-validation within the test set (13 RFE features)**

| Attack | Accuracy | Precision | Recall | F-measure |
|--------|----------|-----------|--------|-----------|
| DoS | 99.74% | 99.69% | 99.71% | 99.70% |
| Probe | 99.09% | 98.67% | 98.47% | 98.57% |
| R2L | 97.46% | 96.69% | 96.09% | 96.38% |
| U2R | 99.65% | 87.54% | 89.54% | 87.73% |

**Trained on KDDTrain+, evaluated on KDDTest+ (from the notebook's confusion matrices)**

| Attack | Accuracy | Precision | Recall |
|--------|----------|-----------|--------|
| DoS | 84.1% | 97.8% | 64.8% |
| Probe | 84.0% | 59.6% | 61.0% |
| R2L | 79.2% | 84.0% | 11.3% |
| U2R | 99.4% | 75.0% | 22.4% |

The gap between the two tables shows how much harder it is to generalise
from the training set to unseen test traffic than to score well within a
single dataset. NSL-KDD's test set contains attack types that do not appear in training.

## Known Limitations
- Cross-validation was run on the test set, so those scores do not measure
  generalisation from training to unseen data
- Feature scalers were fit separately on the test data instead of reusing
  the training scalers
- The one-hot encoder was refit on the test data, so test columns are not
  guaranteed to line up with the training columns
- Rare classes (R2L, U2R) have low recall, which matters most for an
  intrusion detector
- Only one model type (decision tree) and no hyperparameter tuning

## Future Improvements
- Fit all encoders and scalers on training data only, using a scikit-learn `Pipeline`
- Evaluate with a proper train/validation/test split and report per-class recall
- Compare Random Forest, Gradient Boosting and XGBoost
- Handle class imbalance (class weights or SMOTE)
- Build a single multi-class model instead of four binary ones

## Getting Started
1. Install dependencies: `pip install pandas numpy scikit-learn jupyter`
2. Keep the notebook and both CSV files in the same folder
3. Open `ISM PROJECT.ipynb` and run all cells

## Tech Stack
Python 3.10, pandas, NumPy, scikit-learn, Jupyter Notebook
