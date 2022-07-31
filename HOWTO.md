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

time ./runstable.sh -f=$frameworks -b=automl_config_docker -c=1h4c -m=docker 2>&1 | tee main.log
```

## Concatenate all results

```bash
find ./results -name 'results.csv' -exec python concat_csv.py results.csv {} +
```

## Results so far

- `local`
  | id | task | framework | constraint | fold | type | result | metric | mode | version | params | app_version | utc | duration | training_duration | predict_duration | models_count | seed | info | acc | auc | balacc | logloss |
  |---------|---------|-------------------|------------|------|--------|--------|--------|-------|---------|--------|-----------------------------------------------------------------------------------------------------|---------------------|----------|-------------------|------------------|--------------|------------|------|----------|-------|--------|----------|
  | teddata | teddata | flaml | 1h4c | 0 | binary | 0.895 | auc | local | 1.0.8 | | "dev [https://github.com/ucl-medical-genomics/clinical_automl_benchmark, automlbenchmark, 5212a9d]" | 2022-07-19T08:37:37 | 81.3 | 60.7 | 0.4 | 8 | 871770619 | | 0.77551 | 0.895 | 0.775 | 0.431857 |
  | teddata | teddata | constantpredictor | 1h4c | 0 | binary | 0.5 | auc | local | 0.24.2 | | "dev [https://github.com/ucl-medical-genomics/clinical_automl_benchmark, automlbenchmark, 5212a9d]" | 2022-07-19T08:41:07 | 4.8 | 0.0002 | 0.0001 | 1 | 1694040926 | | 0.510204 | 0.5 | 0.5 | 0.692959 |
  | teddata | teddata | lightautoml | 1h4c | 0 | binary | 1 | auc | local | 0.3.6 | | "dev [https://github.com/ucl-medical-genomics/clinical_automl_benchmark, automlbenchmark, 5212a9d]" | 2022-07-19T08:45:06 | 150.5 | 89.8 | 14.8 | 1 | 289279427 | | 1 | 1 | 1 | 0.576326 |

- `docker` after dropping 1st column (`id`)

  Increased `max_runtime_seconds` from 60 to 200s, it stabled reproducibility. Now 7 out of 19 frameworks worked.

  | **id**      | **task** | **framework**     | **constraint** | **fold** | **type** | **result** | **metric** | **mode** | **version**        | **params**             | **app_version**    | **utc**             | **duration** | **training_duration** | **predict_duration** | **models_count** | **seed**   | **info**                                                                                                                                                                                                    | **acc**                                          | **auc** | **balacc** | **logloss** | \*\*\*\* | \*\*\*\* |
  | ----------- | -------- | ----------------- | -------------- | -------- | -------- | ---------- | ---------- | -------- | ------------------ | ---------------------- | ------------------ | ------------------- | ------------ | --------------------- | -------------------- | ---------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------- | ---------- | ----------- | -------- | -------- |
  | **teddata** | teddata  | TPOT              | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 0.11.7             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:21:46 | 348.3        | 336.7                 | 0.01                 | 32.0             | 1896731080 |                                                                                                                                                                                                             | 1.0                                              | 1.0     | 1.0        | 0.267379    |          |          |
  | **teddata** | teddata  | H2OAutoML         | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 3.36.1.3           |                        | "dev [NA, NA, NA]" | 2022-07-30T05:25:34 | 224.2        | 204.0                 | 0.4                  | 13.0             | 757804959  |                                                                                                                                                                                                             | 0.979592                                         | 1.0     | 0.98       | 0.124666    |          |          |
  | **teddata** | teddata  | AutoGluon         | 1h4c           | 0        | binary   |            | auc        | docker   | 0.5.2              |                        | "dev [NA, NA, NA]" | 2022-07-30T05:25:41 | 1.9          |                       |                      |                  | 1651738076 | CalledProcessError: Command '/bench/frameworks/AutoGluon/venv/bin/python -W ignore /bench/frameworks/AutoGluon/exec.py' returned non-zero exit status 1.                                                    |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | autoxgboost       | 1h4c           | 0        | binary   |            | auc        | docker   | 0.0.0.9000#9086316 |                        | "dev [NA, NA, NA]" | 2022-07-30T05:25:49 | 4.3          |                       |                      |                  | 1997826020 | "CalledProcessError: Command 'Rscript --vanilla -e "".libPaths('/bench/frameworks/autoxgboost/lib'); source('/bench/frameworks/autoxgboost/exec.R'); run('/input/test_data/differentiate_cancer_train.csv…" |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | flaml             | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 1.0.8              |                        | "dev [NA, NA, NA]" | 2022-07-30T05:29:21 | 208.3        | 200.5                 | 0.4                  | 13.0             | 1510859588 |                                                                                                                                                                                                             | 1.0                                              | 1.0     | 1.0        | 0.366098    |          |          |
  | **teddata** | teddata  | hyperoptsklearn   | 1h4c           | 0        | binary   |            | auc        | docker   | latest             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:29:32 | 6.8          |                       |                      |                  | 1164947945 | NoResultError: Only one class present in y_true. ROC AUC score is not defined in that case.                                                                                                                 |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | mljarsupervised   | 1h4c           | 0        | binary   |            | auc        | docker   | 0.11.2             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:29:40 | 4.1          |                       |                      |                  | 538611131  | CalledProcessError: Command '/bench/frameworks/mljarsupervised/venv/bin/python -W ignore /bench/frameworks/mljarsupervised/exec.py' returned non-zero exit status 1.                                        |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | mlr3automl        | 1h4c           | 0        | binary   |            | auc        | docker   | 0.0.0.9000#f667900 |                        | "dev [NA, NA, NA]" | 2022-07-30T05:29:49 | 4.2          |                       |                      |                  | 153344726  | "CalledProcessError: Command 'Rscript --vanilla -e "".libPaths('/bench/frameworks/mlr3automl/lib'); source('/bench/frameworks/mlr3automl/exec.R'); run('/input/test_data/differentiate_cancer_train.csv'    | …"                                               |         |            |             |          |          |
  | **teddata** | teddata  | RandomForest      | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 1.0.2              | {'n_estimators': 2000} | "dev [NA, NA, NA]" | 2022-07-30T05:30:21 | 28.3         | 21.7                  | 0.4                  | 2000.0           | 230486926  |                                                                                                                                                                                                             | 1.0                                              | 1.0     | 1.0        | 0.18348     |          |          |
  | **teddata** | teddata  | autosklearn       | 1h4c           | 0        | binary   |            | auc        | docker   | 0.14.7             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:30:30 | 5.3          |                       |                      |                  | 1700088810 | CalledProcessError: Command '/bench/frameworks/autosklearn/venv/bin/python -W ignore /bench/frameworks/autosklearn/exec.py' returned non-zero exit status 1.                                                |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | constantpredictor | 1h4c           | 0        | binary   | 0.5        | auc        | docker   | 0.24.2             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:30:37 | 2.5          | 0.0002                | 6e-05                | 1.0              | 1865484292 |                                                                                                                                                                                                             | 0.510204                                         | 0.5     | 0.5        | 0.693123    |          |          |
  | **teddata** | teddata  | GAMA              | 1h4c           | 0        | binary   |            | auc        | docker   | 21.0.1             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:30:48 | 7.1          |                       |                      |                  | 1284338721 | NoResultError: 'ignore'                                                                                                                                                                                     |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | MLNet             | 1h4c           | 0        | binary   |            | auc        | docker   | 16.8.3             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:43:32 | 760.0        |                       |                      |                  | 1616757221 | TimeoutError: Interrupting thread MainThread [ident=139847097177920] after 700s timeout.                                                                                                                    |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | oboe              | 1h4c           | 0        | binary   |            | auc        | docker   | latest             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:43:39 | 3.2          |                       |                      |                  | 1733069952 | CalledProcessError: Command '/bench/frameworks/oboe/venv/bin/python -W ignore /bench/frameworks/oboe/exec.py' returned non-zero exit status 1.                                                              |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | ranger            | 1h4c           | 0        | binary   |            | auc        | docker   | 0.14.1             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:43:45 | 1.7          |                       |                      |                  | 1865424003 | "CalledProcessError: Command 'Rscript --vanilla -e ""source('/bench/frameworks/ranger/exec.R'); run('/input/test_data/differentiate_cancer_train.csv'                                                       | '/input/test_data/differentiate_cancer_test.csv' | …"      |            |             |          |          |
  | **teddata** | teddata  | TunedRandomForest | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 1.0.2              | {'n_estimators': 2000} | "dev [NA, NA, NA]" | 2022-07-30T05:47:15 | 206.0        | 200.0                 | 0.1                  | 680.0            | 1882717959 |                                                                                                                                                                                                             | 1.0                                              | 1.0     | 1.0        | 0.19032     |          |          |
  | **teddata** | teddata  | AutoWEKA          | 1h4c           | 0        | binary   |            | auc        | docker   | latest             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:51:57 | 278.2        |                       |                      |                  | 1437477452 | NoResultError: AutoWEKA failed producing any prediction.                                                                                                                                                    |                                                  |         |            |             |          |          |
  | **teddata** | teddata  | DecisionTree      | 1h4c           | 0        | binary   | 1.0        | auc        | docker   | 0.24.2             |                        | "dev [NA, NA, NA]" | 2022-07-30T05:52:04 | 3.5          | 0.3                   | 0.0005               | 1.0              | 1317944968 |                                                                                                                                                                                                             | 1.0                                              | 1.0     | 1.0        | 9.99201e-16 |          |          |
  | **teddata** | teddata  | lightautoml       | 1h4c           | 0        | binary   |            | auc        | docker   | 0.3.6              |                        | "dev [NA, NA, NA]" | 2022-07-30T05:52:12 | 3.9          |                       |                      |                  | 1529315974 | CalledProcessError: Command '/bench/frameworks/lightautoml/venv/bin/python -W ignore /bench/frameworks/lightautoml/exec.py' returned non-zero exit status 1.                                                |                                                  |         |            |             |          |          |
