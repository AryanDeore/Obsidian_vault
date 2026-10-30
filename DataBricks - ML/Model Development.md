### MLflow:
mlflow.sklearn`.autolog` - mlflow autologs sklearns outputs
mlflow.data`.load_delta`- grab feature table as mlflow dataset object

### Optuna
Hyperparameter optimization engine

**Study**: optimization session containing multiple trials
**Trial**: 1 evaluation of the objective function with 1 hyper-parameter set
**Objective function**: The metric whose values are to be optimized ie loss function or error.

**Search Space**: specifies range of values each hyper-parameter can take
**Sampling strategy**: determine how the next step of hyper-parameter is chosen. eg: grid search, random search, TPE
**Pruning strategy**: Stop trials early based on intermediate results.

- use `suggest_int` and `suggest_categorical` while defining hyper-parameter search space.
- Setting `n_jobs = -1` parallelizes runs across all available cpu cores


