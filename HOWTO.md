# Usage Running Teddy's data

Run in `overdrive`, with `docker` option and under `screen` or `tmux` for safety.

## Set environment

Install python modules needed byt `runbenchmark.py`

```bash
python3 -m venv ./venv
source venv/bin/activate
python3 -m pip install -r requirements.txt
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
