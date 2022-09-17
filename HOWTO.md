# Usage Running Teddy's data

Run in `overdrive`, with `docker` option and under `screen` or `tmux` for safety.

Rationale:

- `docker` for production
- `local` for debugging

## Set environment

Install python modules needed by `runbenchmark.py`

```bash
python3 -m venv ./venv
source venv/bin/activate
python3 -m pip install -r requirements.txt
```

## Get `test_data/`

Copy input CSVs:

```bash
cp -a ../clinical_automl_benchmark/test_data/ .
```

## Run with `local`

Just for an example.

```bash
python3 runbenchmark.py TPOT automl_config 1h4c -m local
```

## Run with `docker`

```bash
yes | python3 runbenchmark.py TPOT automl_config_docker 1h4c -m docker -i .
```

## Run all frameworks

```bash
frameworks="TPOT H2OAutoML AutoGluon autoxgboost flaml hyperoptsklearn mljarsupervised mlr3automl RandomForest autosklearn constantpredictor GAMA MLNet oboe ranger TunedRandomForest AutoWEKA DecisionTree lightautoml"

time ./runstable.sh -f=$frameworks -b=automl_config_docker -c=1h4c -s=force -m=docker 2>&1 | tee main_docker.log
```

## Concatenate all results

```bash
find ./results -name 'results.csv' -exec python concat_csv.py results.csv {} +
```

## Results so far

- `docker` after dropping 1st column (`id`)

  Increased `max_runtime_seconds` from 60 to 200s, it stabled reproducibility. Fixed several docker issues. Now 11 out of 19 frameworks worked.

  **`| contraint = 1h4c | fold = 0 | type = binary | metric = auc | mode = docker | app_version = NA |`**

  | **framework**         | **result** | **metric** | **version**        | **params**             | **utc**             | **duration** | **training_duration** | **predict_duration** | **models_count** | **seed**   | **info**                                                                                                                                                                                                    | **acc**  | **auc** | **balacc** | **logloss** | **models_ensemble_count** |
  | --------------------- | ---------- | ---------- | ------------------ | ---------------------- | ------------------- | ------------ | --------------------- | -------------------- | ---------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----------- | ------------------------- |
  | **TPOT**              | 1.0        | auc        | 0.11.7             |                        | 2022-09-16T21:45:57 | 342.1        | 331.7                 | 0.01                 | 32.0             | 1270515697 |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.376394    |                           |
  | **H2OAutoML**         | 1.0        | auc        | 3.36.1.5           |                        | 2022-09-16T21:56:01 | 217.6        | 201.9                 | 0.3                  | 13.0             | 1347614862 |                                                                                                                                                                                                             | 0.897959 | 1.0     | 0.9        | 0.00256132  |                           |
  | **AutoGluon**         | 1.0        | auc        | 0.5.2              |                        | 2022-09-16T22:08:13 | 209.4        | 204.1                 | 1.0                  | 8.0              | 434704218  |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.224987    | 2.0                       |
  | **autoxgboost**       |            | auc        | 0.0.0.9000#9086316 |                        | 2022-09-16T22:29:21 | 3.9          |                       |                      |                  | 695446268  | "CalledProcessError: Command 'Rscript --vanilla -e "".libPaths('/bench/frameworks/autoxgboost/lib'); source('/bench/frameworks/autoxgboost/exec.R'); run('/input/test_data/differentiate_cancer_train.csv…" |          |         |            |             |                           |
  | **flaml**             | 0.995      | auc        | 1.0.12             |                        | 2022-09-16T22:38:35 | 208.8        | 202.0                 | 0.3                  | 10.0             | 491871903  |                                                                                                                                                                                                             | 0.938776 | 0.995   | 0.938333   | 0.492482    |                           |
  | **hyperoptsklearn**   |            | auc        | latest             |                        | 2022-09-16T22:44:47 | 4.1          |                       |                      |                  | 1562658159 | NoResultError: Only one class present in y_true. ROC AUC score is not defined in that case.                                                                                                                 |          |         |            |             |                           |
  | **mljarsupervised**   | 1.0        | auc        | 0.11.3             |                        | 2022-09-16T22:56:35 | 401.8        | 388.1                 | 0.6                  | 6.0              | 1556328829 |                                                                                                                                                                                                             | 0.653061 | 1.0     | 0.645833   | 9.99201e-16 |                           |
  | **mlr3automl**        |            | auc        | 0.0.0.9000#f667900 |                        | 2022-09-16T23:20:37 | 3.7          |                       |                      |                  | 500736702  | "CalledProcessError: Command 'Rscript --vanilla -e "".libPaths('/bench/frameworks/mlr3automl/lib'); source('/bench/frameworks/mlr3automl/exec.R'); run('/input/test_data/differentiate_cancer_train.csv'…"  |          |         |            |             |                           |
  | **RandomForest**      | 1.0        | auc        | 1.0.2              | {'n_estimators': 2000} | 2022-09-16T23:25:04 | 26.9         | 21.3                  | 0.3                  | 2000.0           | 350634934  |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.183562    |                           |
  | **autosklearn**       | 1.0        | auc        | 0.14.7             |                        | 2022-09-16T23:34:50 | 259.2        | 201.5                 | 6.5                  | 30.0             | 455233695  |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.232862    |                           |
  | **constantpredictor** | 0.5        | auc        | 0.24.2             |                        | 2022-09-16T23:38:01 | 2.4          | 0.0001                | 6e-05                | 1.0              | 923632748  |                                                                                                                                                                                                             | 0.510204 | 0.5     | 0.5        | 0.693123    |                           |
  | **GAMA**              |            | auc        | 21.0.1             |                        | 2022-09-16T23:41:52 | 6.1          |                       |                      |                  | 1933950374 | NoResultError: 'ignore'                                                                                                                                                                                     |          |         |            |             |                           |
  | **MLNet**             |            | auc        | 16.8.3             |                        | 2022-09-17T00:02:26 | 760.1        |                       |                      |                  | 711012818  | TimeoutError: Interrupting thread MainThread [ident=140236646168384] after 700s timeout.                                                                                                                    |          |         |            |             |                           |
  | **oboe**              |            | auc        | latest             |                        | 2022-09-17T00:09:19 | 3.3          |                       |                      |                  | 1736021790 | CalledProcessError: Command '/bench/frameworks/oboe/venv/bin/python -W ignore /bench/frameworks/oboe/exec.py' returned non-zero exit status 1.                                                              |          |         |            |             |                           |
  | **ranger**            |            | auc        | stable             |                        | 2022-09-17T00:16:28 | 1.6          |                       |                      |                  | 1739105207 | "CalledProcessError: Command 'Rscript --vanilla -e "source('/bench/frameworks/ranger/exec.R'); run('/input/test_data/differentiate_cancer_train.csv'…"                                                      |          |         |            |             |                           |
  | **TunedRandomForest** | 1.0        | auc        | 1.0.2              | {'n_estimators': 2000} | 2022-09-17T00:25:30 | 205.8        | 200.0                 | 0.2                  | 970.0            | 263079772  |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.184835    |                           |
  | **AutoWEKA**          |            | auc        | latest             |                        | 2022-09-17T00:36:15 | 276.7        |                       |                      |                  | 213918073  | NoResultError: AutoWEKA failed producing any prediction.                                                                                                                                                    |          |         |            |             |                           |
  | **DecisionTree**      | 1.0        | auc        | 0.24.2             |                        | 2022-09-17T00:39:53 | 3.5          | 0.3                   | 0.0003               | 1.0              | 2039352633 |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 9.99201e-16 |                           |
  | **lightautoml**       | 1.0        | auc        | 0.3.7              |                        | 2022-09-17T00:48:59 | 167.1        | 144.3                 | 8.6                  | 1.0              | 1773285132 |                                                                                                                                                                                                             | 1.0      | 1.0     | 1.0        | 0.55931     |                           |
