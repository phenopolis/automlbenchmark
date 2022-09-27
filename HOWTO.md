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

  Increased `max_runtime_seconds` from 60 to 600, it stabled reproducibility. Fixed several docker issues. Now 17 out of 20 frameworks worked.

  **`| constraint = 1h4c | fold = 0 | type = binary | metric = auc | mode = docker | app_version = NA |`**

  | **framework**         | **result** | **version**        | **params**             | **duration** | **training_duration** | **predict_duration** | **models_count** | **seed**   | **info**                                                                                                              | **acc**  | **auc**  | **balacc** | **logloss** | **models_ensemble_count** |
  | --------------------- | ---------- | ------------------ | ---------------------- | ------------ | --------------------- | -------------------- | ---------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- | -------- | -------- | ---------- | ----------- | ------------------------- |
  | **TPOT**              | 1.0        | 0.11.7             |                        | 737.5        | 725.5                 | 0.01                 | 64.0             | 1101571440 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 4.7852e-10  |                           |
  | **H2OAutoML**         | 1.0        | 3.38.0.1           |                        | 418.2        | 402.6                 | 0.2                  | 16.0             | 1407531825 |                                                                                                                       | 0.591837 | 1.0      | 0.6        | 0.198849    |                           |
  | **AutoGluon**         | 1.0        | 0.5.2              |                        | 410.9        | 404.1                 | 0.07                 | 8.0              | 1669703677 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.224987    | 2.0                       |
  | **autoxgboost**       | 0.996667   | 0.0.0.9000#9086316 |                        | 469.6        | 414.2                 | 0.6                  |                  | 441430030  |                                                                                                                       | 0.959184 | 0.996667 | 0.958333   | 0.167646    |                           |
  | **flaml**             | 1.0        | 1.0.12             |                        | 410.8        | 401.4                 | 1.2                  | 15.0             | 85168030   |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.220788    |                           |
  | **hyperoptsklearn**   | 1.0        | latest             |                        | 8.1          | 1.6                   | 0.001                | 10               | 85982072   |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **mljarsupervised**   | 1.0        | 0.11.3             |                        | 264.7        | 249.5                 | 1.5                  | 6.0              | 1152325447 |                                                                                                                       | 0.653061 | 1.0      | 0.645833   | 9.99201e-16 |                           |
  | **mlr3automl**        | 1.0        | 0.0.0.9000#f667900 |                        | 361.8        | 352.7                 | 2.6                  |                  | 455053604  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.396167    |                           |
  | **RandomForest**      | 1.0        | 1.1.2              | {'n_estimators': 2000} | 12.0         | 5.0                   | 0.2                  | 2000.0           | 604959814  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.185356    |                           |
  | **autosklearn**       | 1.0        | 0.15.0             |                        | 446.6        | 432.5                 | 1.0                  | 31.0             | 868244422  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.223702    |                           |
  | **constantpredictor** | 0.5        | 0.24.2             |                        | 3.4          | 0.0002                | 6e-05                | 1.0              | 502615866  |                                                                                                                       | 0.510204 | 0.5      | 0.5        | 0.693123    |                           |
  | **GAMA**              | 1.0        | 22.0.1.dev         |                        | 379.1        | 370.7                 | 0.03                 | 50.0             | 790931519  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.000955536 |                           |
  | **MLNet**             | 1.0        | 16.8.3             |                        | 1011.0       | 709.5                 | 298.6                | 10.0             | 2104352482 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.0863285   |                           |
  | **oboe**              |            | latest             |                        | 7.5          |                       |                      |                  | 613776517  | "NoResultError: shape mismatch: value array of shape (35,) could not be broadcast to indexing result of shape (35,1)" |          |          |            |             |                           |
  | **ranger**            | 1.0        | 0.0.0.9000#f667900 |                        | 61.1         | 13.9                  | 0.2                  |                  | 1239691598 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.201067    |                           |
  | **TunedRandomForest** | 1.0        | 1.1.2              | {'n_estimators': 2000} | 406.4        | 399.6                 | 0.1                  | 990.0            | 1850023945 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.188997    |                           |
  | **AutoWEKA**          | 1.0        | latest             |                        | 1087.9       | 1085.6                |                      |                  | 180145670  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.00276314  |                           |
  | **DecisionTree**      | 1.0        | 0.24.2             |                        | 4.4          | 0.3                   | 0.0003               | 1.0              | 804902253  |                                                                                                                       | 1.0      | 1.0      | 1.0        | 9.99201e-16 |                           |
  | **lightautoml**       | 1.0        | 0.3.7.1            |                        | 272.2        | 223.8                 | 32.2                 | 1.0              | 1725320065 |                                                                                                                       | 1.0      | 1.0      | 1.0        | 0.320414    |                           |
  | **MLPlanWEKA**        |            | stable             | {'\_backend': 'weka'}  | 2.4          |                       |                      |                  | 1915139012 | NoResultError: list index out of range                                                                                |          |          |            |             |                           |

## The failing frameworks

From 20, 2 have failed and here is the summary for them after fixing the code as much as possible:

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

2. `MLPlanWEKA`.

   ```log
   ...
   unzip:  cannot find zipfile directory in one of /bench/frameworks/MLPlan/lib/mlplan.zip or
           /bench/frameworks/MLPlan/lib/mlplan.zip.zip, and cannot find /bench/frameworks/MLPlan/lib/mlplan.zip.ZIP, period.
   find: ‘/bench/frameworks/MLPlan/lib/mlplan/*.jar’: No such file or directory
   ...
   Running cmd `/bench/frameworks/MLPlan/venv/bin/python -W ignore /bench/frameworks/MLPlan/exec.py`
   ERROR:frameworks.shared.callee:list index out of range
   Traceback (most recent call last):

     File "/bench/frameworks/shared/callee.py", line 70, in call_run

       result = run_fn(ds, config)

     File "/bench/frameworks/MLPlan/exec.py", line 15, in run

       jar_file = glob.glob(f"{os.path.dirname(__file__)}/lib/mlplan/mlplan-cli*.jar")[0]

   IndexError: list index out of range
   ...
   ```
