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
frameworks="TPOT H2OAutoML AutoGluon autoxgboost flaml hyperoptsklearn mljarsupervised mlr3automl RandomForest autosklearn constantpredictor GAMA:latest MLNet oboe ranger TunedRandomForest AutoWEKA DecisionTree lightautoml mlplanweka"

time ./runstable.sh -f=$frameworks -b=automl_config_docker -c=1h4c -s=force -m=docker 2>&1 | tee main_docker.log
```

## Concatenate all results

```bash
find ./results -name 'results.csv' -exec python concat_csv.py results.csv {} +
```

## Results so far

- `docker` after dropping 1st column (`id`)

  Increased `max_runtime_seconds` from 60 to 600, it stabled reproducibility. Fixed several docker issues, some with help of AMLB developers, by testing and given feedback. Also, input data converted to ARFF format and shuffled, so `hyperoptsklearn` got to work, however `MLNet`\* gets `0.4` instead of `1.0` for `result`. Now 19 out of 20 frameworks worked.

  **`| constraint = 1h4c | fold = 0 | type = binary | metric = auc | mode = docker | app_version = NA |`**

  | **framework**         | **result** | **version**        | **params**             | **duration** | **training_duration** | **predict_duration** | **models_count** | **seed**   | **info**                                                                                                              | **acc**  | **auc**  | **balacc** | **logloss** | **models_ensemble_count** |
  | --------------------- | ---------- | ------------------ | ---------------------- | ------------ | --------------------- | -------------------- | ---------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- | -------- | -------- | ---------- | ----------- | ------------------------- |
  | **TPOT**              | 1.0        | 0.11.7             |                        | 639.0        | 626.6                 | 0.05                 | 32.0             | 909778066  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.000469174 |                           |
  | **H2OAutoML**         | 1.0        | 3.38.0.1           |                        | 620.3        | 603.6                 | 0.3                  | 20.0             | 871014118  |                                                                                                                       | 0.979592 | 1.0      | 0.98       | 0.0393543   |                           |
  | **AutoGluon**         | 1.0        | 0.5.2              |                        | 615.3        | 607.1                 | 0.07                 | 10.0             | 519073248  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.262019    | 2.0                       |
  | **autoxgboost**       | 0.986667   | 0.0.0.9000#9086316 |                        | 644.2        | 596.5                 | 0.3                  |                  | 621596810  |                                                                                                                       | 0.938776 | 0.986667 | 0.94       | 0.305582    |                           |
  | **flaml**             | 1.0        | 1.0.12             |                        | 610.4        | 600.3                 | 1.2                  | 14.0             | 1930816097 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.45442     |                           |
  | **hyperoptsklearn**   | 1.0        | latest             |                        | 146.1        | 139.5                 | 0.08                 | 10.0             | 478399822  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **mljarsupervised**   | 1.0        | 0.11.3             |                        | 206.9        | 190.5                 | 1.6                  | 6.0              | 1209644479 |                                                                                                                       | 0.571429 | 1.0      | 0.5625     | 2.22058e-12 |                           |
  | **mlr3automl**        | 1.0        | 0.0.0.9000#f667900 |                        | 522.2        | 512.8                 | 2.5                  |                  | 1715902538 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.414777    |                           |
  | **RandomForest**      | 1.0        | 1.1.2              | {'n_estimators': 2000} | 12.5         | 4.9                   | 0.2                  | 2000.0           | 1218348910 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.185046    |                           |
  | **autosklearn**       | 1.0        | 0.15.0             |                        | 638.1        | 623.5                 | 0.9                  | 29.0             | 472747926  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.249879    |                           |
  | **constantpredictor** | 0.5        | 0.24.2             |                        | 3.4          | 0.0002                | 6e-05                | 1.0              | 1448423884 |                                                                                                                       | 0.510204 | 0.5      | 0.5        | 0.693123    |                           |
  | **GAMA**              | 1.0        | 22.0.1.dev         |                        | 559.7        | 550.4                 | 0.07                 | 50.0             | 1472092503 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **MLNet**             | 0.4\*      | 16.8.3             |                        | 1214.8       | 914.0                 | 297.9                | 16.0             | 1074692474 |                                                                                                                       | 0.469388 | 0.4      | 0.469167   | 1.54912     |                           |
  | **oboe**              |            | latest             |                        | 7.8          |                       |                      |                  | 1332156373 | "NoResultError: shape mismatch: value array of shape (35,) could not be broadcast to indexing result of shape (35,1)" |          |          |            |             |                           |
  | **ranger**            | 1.0        | 0.0.0.9000#f667900 |                        | 61.0         | 14.0                  | 0.2                  |                  | 1132688834 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.203446    |                           |
  | **TunedRandomForest** | 1.0        | 1.1.2              | {'n_estimators': 2000} | 607.2        | 599.9                 | 0.2                  | 1510.0           | 2021248236 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.193447    |                           |
  | **AutoWEKA**          | 1.0        | latest             |                        | 1304.4       | 1302.0                |                      |                  | 805451357  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **DecisionTree**      | 1.0        | 0.24.2             |                        | 4.4          | 0.3                   | 0.0003               | 1.0              | 1244593699 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **lightautoml**       | 1.0        | 0.3.7.1            |                        | 455.4        | 405.2                 | 33.0                 | 1.0              | 143211157  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.312956    |                           |
  | **MLPlanWEKA**        | 1.0        | 0.0.2              | {'\_backend': 'weka'}  | 498.6        | 495.6                 | 171.0                | 47.0             | 1578132338 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |

## The failing frameworks

From 20, only one is failing :

1. `oboe`. See [issue](https://github.com/openml/automlbenchmark/issues/496) for more. Oboe has been inactive for a while but it may be back.

   ```log
   ...
   **** Oboe [latest] ****

   INFO:__main__:Running oboe with a maximum time of 200s on 4 cores.
   WARNING:__main__:We completely ignore the advice to optimize towards metric: auc.
   ERROR:frameworks.shared.callee:shape mismatch: value array of shape (35,) could not be broadcast to indexing result of shape (35,1)
   multiprocessing.pool.RemoteTraceback:

   """

   Traceback (most recent call last):

     File "/usr/lib/python3.7/multiprocessing/pool.py", line 121, in worker

       result = (True, func(*args, **kwds))

     File "/bench/frameworks/oboe/lib/oboe/oboe/model.py", line 102, in kfold_fit_validate

       y_predicted[test_idx] = model.predict(x_te)

   ValueError: shape mismatch: value array of shape (35,) could not be broadcast to indexing result of shape (35,1)
   ...
   ```
