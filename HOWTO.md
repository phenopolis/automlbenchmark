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

- `docker`
  | id | task | framework | constraint | fold | type | result | metric | mode | version | params | app_version | utc | duration | training_duration | predict_duration | models_count | seed | info | acc | auc | balacc | logloss |
  | ------- | ------- | ----------------- | ---------- | ---- | ------ | ------ | ------ | ------ | -------- | ------ | ------------------ | ------------------- | -------- | ----------------- | ---------------- | ------------ | ---------- | ---- | -------- | --- | -------- | ---------- |
  | teddata | teddata | H2OAutoML | 1h4c | 0 | binary | 1.0 | auc | docker | 3.36.1.3 | | "dev [NA, NA, NA]" | 2022-07-22T13:40:10 | 80.7 | 63.6 | 0.3 | 14.0 | 378942758 | | 0.979592 | 1.0 | 0.98 | 0.00836138 |
  | teddata | teddata | flaml | 1h4c | 0 | binary | 1.0 | auc | docker | 1.0.8 | | "dev [NA, NA, NA]" | 2022-07-22T13:41:32 | 67.4 | 60.5 | 0.3 | 13.0 | 1739355544 | | 0.979592 | 1.0 | 0.979167 | 0.423903 |
  | teddata | teddata | constantpredictor | 1h4c | 0 | binary | 0.5 | auc | docker | 0.24.2 | | "dev [NA, NA, NA]" | 2022-07-22T13:42:07 | 2.4 | 0.0002 | 6e-05 | 1.0 | 565229265 | | 0.510204 | 0.5 | 0.5 | 0.693123 |
