# INX Future Inc. – Employee Performance Analysis (IABAC project 10281)

## How to run
1. `pip install -r requirements.txt`
2. Run the notebooks **in this order** (each one uses relative paths and can be run from its own folder):
   1. `src/Data Processing/data_processing.ipynb`
   2. `src/Data Processing/data_exploratory_analysis.ipynb`
   3. `src/models/train_&_predict_model.ipynb` (about 2-4 minutes; trains, compares and tunes the models, then evaluates the selected one)
   4. `src/visualization/visualize.ipynb`

All notebooks are already executed, so outputs can be read without running anything.

## Folder guide
| Folder | Contents |
|---|---|
| `Project Summary/Requirement` | IABAC project brief (PDF) and requirements summary |
| `Project Summary/Analysis` | Algorithm selection and decisions (`Analysis.md`) |
| `Project Summary/Summary` | Full documentation, findings and recommendations (`Project_Documentation.md`) |
| `data/raw` | Original dataset, unchanged |
| `data/processed` | Cleaned data, encoded preview, train and test sets |
| `data/external` | Not used (no external data) |
| `src/Data Processing` | Data processing and EDA notebooks |
| `src/models` | Model training, comparison and evaluation notebook |
| `src/visualization` | Summary visuals notebook |
| `reports` | Result tables, employee risk scores, feature importance, model metrics |
| `references` | Data dictionary |

## Important note
Model predictions are for support and review only. Do not use them as the sole basis for hiring, rejecting or penalising anyone. A model using only candidate information (hiring model) performs worse than simply guessing "rating 3" for everyone, so it is not recommended for decisions.