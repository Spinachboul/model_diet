# API reference

## API stability and support tiers

Model Diet is currently pre-1.0. The table describes the support intent for the interfaces as they exist today; it does not imply that every importable module is a supported extension point.

| Tier | Import path | Status |
| --- | --- | --- |
| Stable core | `modeldiet.profile`, `modeldiet.ProfileResult` | Supported primary API. |
| Measurement extension API | `modeldiet.profiling.BaseMeasurement`, `LatencyMeasurement`, `ProcessRSSMeasurement`, `SerializationMeasurement`, `TracedAllocationMeasurement` | Supported strategy classes. Use the `modeldiet.profiling` imports shown below. |
| Result records | `modeldiet.result.MetricDefinition`, `MetricValue`, `MeasurementError`, `MeasurementResult`, `ProfileResult` | Supported data structures at these import paths. `ProfileResult` is also exported from `modeldiet`. |
| Optional benchmark tooling | `modeldiet.benchmark.run_repeated_benchmark` | Provisional pre-1.0 tooling. It runs bundled demonstration data and estimators, and is separate from general-purpose profiling. |
| Legacy/compatibility | `modeldiet.profiling.benchmark.run_repeated_benchmark`; `modeldiet.profiling.isolated_runner` | Compatibility paths for optional benchmark tooling; no warning is issued on import. |
| Deprecated compatibility | `modeldiet.model.Model`; `modeldiet.models.lr.LogisticRegressionModel`; `modeldiet.models.rf.RandomForestModel`; `modeldiet.models.xgb.XGBoostModel`; `modeldiet.report.Report`; `modeldiet.driver.main` | Existing imports continue to work. Instantiating the model/wrapper/report classes or calling `driver.main()` emits `DeprecationWarning`. Removal will not occur before 1.0.0. |
| Internal implementation | `modeldiet.api`, `modeldiet.profiling.orchestrator`; `modeldiet.benchmark.worker.create_model`, `load_dataset`, `run_model`, `main`; `modeldiet.benchmark.statistics.calculate_statistics`, `summarize_runs`; `environment.collect_environment`, `EnvironmentMeasurement`; private helpers such as `_definition` | Implementation details without a compatibility promise. |

`profile()` and `ProfileResult` are the recommended way to profile a user's own fitted model. Import built-in strategies from `modeldiet.profiling`. Import result record types other than `ProfileResult` from `modeldiet.result`; these types are not root-level exports. Import the benchmark helper from `modeldiet.benchmark`. The old `modeldiet.profiling.benchmark` path remains as a compatibility shim.

Benchmark implementation modules now live under `modeldiet.benchmark`: the worker is `modeldiet.benchmark.worker`, legacy model/report implementations are under `modeldiet.benchmark.model` and `modeldiet.benchmark.report`, and demonstration estimators are under `modeldiet.benchmark.models`. Previous `modeldiet.model`, `modeldiet.report`, `modeldiet.models.*`, and `modeldiet.profiling.isolated_runner` import paths remain as thin shims. Both `python -m modeldiet.benchmark.worker` and the legacy `python -m modeldiet.profiling.isolated_runner` command are supported.

## Compatibility policy

Model Diet is version `0.x`, so its API is not yet frozen. We will still prefer backward-compatible changes and will call out intentional breaking changes in release notes. Existing users may rely on documented paths and on established importable compatibility interfaces, so pre-1.0 status alone is not a reason to remove them. Where practical, a breaking pre-1.0 change should include a migration note.

For future 1.x releases, supported public imports, constructors, method signatures, result fields, metric identifiers, and units will be preserved across compatible releases. A supported interface will not be removed or renamed without advance deprecation and migration guidance; intentional breaking public API changes require a major version bump. This policy is an intent, not a promise that every platform, model library, or runtime measurement will behave identically.

The legacy `Model`, estimator wrappers, `Report`, and `driver.main()` are deprecated in the 0.1.0 release. Deprecation warnings are emitted when these classes are instantiated or `driver.main()` is called—not when their modules are imported. Their import paths remain usable through the 0.x series; removal will not occur before 1.0.0, with any removal called out in release notes. `run_repeated_benchmark()` and the old benchmark/worker module paths remain available as benchmark compatibility interfaces and are not deprecated by this change.

`ProfileResult` currently uses JSON schema version 1 (`schema_version`). Breaking changes to JSON field names, types, or interpretation must be explicitly versioned with a schema-version change and migration guidance. Adding compatible fields may be possible without changing the schema version, but consumers should ignore fields they do not recognize. Metric identifiers and units are part of the result contract; their meaning must not be silently changed.

`profile()` raises `TypeError` for a model without callable `predict()` or a strategy that is not a `BaseMeasurement`; invalid `on_error`, missing input, and an empty strategy sequence raise `ValueError`. By default, measurement failures raise `RuntimeError`; `on_error="record"` stores the error and continues. Export methods accept a string or `Path`; the parent directory must already exist. These behaviors are described here as current contracts.

Core profiling depends on NumPy and psutil. Scikit-learn, pandas, and XGBoost are optional. The `benchmark` extra provides pandas and scikit-learn for LR/RF benchmark runs; `benchmark-xgboost` provides pandas, scikit-learn, and XGBoost; the `demo` extra includes all three. User models that provide a compatible `predict(X)` method need none of these optional libraries. Install from production PyPI with `python -m pip install modeldiet`; extra installation commands are shown in the [README installation guidance](https://github.com/modeldiet/modeldiet#install) and the [benchmark runner guide](application.md#optional-benchmark-runner).

## Root imports

The supported root imports are intentionally limited:

```python
from modeldiet import ProfileResult, profile
```

`modeldiet.__all__` contains these two names.

## `modeldiet.profile`

```python
profile(
    model,
    X,
    *,
    measurements=None,
    on_error="raise",
    include_environment=False,
)
```

Profile a fitted model and return a `ProfileResult`.

| Argument | Description |
| --- | --- |
| `model` | Fitted object with a callable `predict(X)` method. |
| `X` | Non-empty, two-dimensional NumPy array, pandas DataFrame, list of rows, or tuple of rows. |
| `measurements` | Optional sequence of `BaseMeasurement` instances. If omitted, only `LatencyMeasurement()` is used. Passing a sequence replaces the default; add latency explicitly if desired. Names must be unique. |
| `on_error` | `"raise"` (default) raises `RuntimeError` on measurement failure. `"record"` stores the failure and continues with remaining measurements. |
| `include_environment` | If true, attaches collected host, Python, and selected library metadata. |

Invalid `on_error`, a model without `predict()`, missing input, empty measurements, and non-`BaseMeasurement` strategies raise errors before or during profiling.

## Built-in measurements

Import strategies from `modeldiet.profiling`:

```python
from modeldiet.profiling import (
    BaseMeasurement,
    LatencyMeasurement,
    ProcessRSSMeasurement,
    SerializationMeasurement,
    TracedAllocationMeasurement,
)
```

`LatencyMeasurement(warmup_iterations=1)` accepts a non-negative number of warmup predictions. Other built-in strategies currently use their default behavior and accept no constructor options.

## `ProfileResult`

Import from `modeldiet`:

```python
from modeldiet import ProfileResult
```

The result exposes `schema_version`, `measurements`, `model_class`, `model_module`, `input_type`, `sample_count`, `feature_count`, and optional `environment` attributes. Each measurement entry is a `MeasurementResult` with a `name`, a `metrics` mapping, and an optional `error`. Its `succeeded` property is true when there is no error.

Each metric value is a `MetricValue`; read its payload from `.value` and its `MetricDefinition` from `.definition`. The definition includes `identifier`, `unit`, `description`, and `semantics`.

The result records are defined in `modeldiet.result`:

```python
from modeldiet.result import (
    MeasurementError,
    MeasurementResult,
    MetricDefinition,
    MetricValue,
    ProfileResult,
)
```

The current `ProfileResult` fields are `schema_version`, `measurements`, `model_class`, `model_module`, `input_type`, `sample_count`, `feature_count`, and `environment`. `MeasurementResult` has `name`, `metrics`, and `error`; `MeasurementError` has `exception_type` and `message`; `MetricValue` has `definition` and `value`; `MetricDefinition` has `identifier`, `unit`, `description`, and `semantics`. These field names and meanings are part of the documented result contract; JSON compatibility is governed by the schema-version policy above.

## Export methods

```python
result.to_json(path)
result.to_markdown(path)
```

`path` is a string or `pathlib.Path`. The parent directory must already exist. JSON is indented, ends with a newline, and uses `null` for non-finite float values. Markdown renders measurement metrics and optional environment metadata.

## Custom measurements

Subclass `BaseMeasurement`, set a unique strategy `name`, and implement `measure(model, X=None)` to return a mapping of serializable keys and values. Built-in metric definitions are provided for built-in strategies. Custom values receive generic definitions; Model Diet cannot infer their units or semantics.

## Metric identifiers and units

Built-in result metrics currently use these stable identifiers and units:

| Strategy | Metric key | Identifier | Unit |
| --- | --- | --- | --- |
| `latency` | `mean_ms`, `p50_ms`, `p90_ms`, `p95_ms`, `p99_ms` | `latency.mean`, `latency.p50`, `latency.p90`, `latency.p95`, `latency.p99` | `ms` |
| `latency` | `sample_count` | `latency.sample_count` | `count` |
| `python_traced_allocation` | `traced_peak_inference_allocation_mb` | `python_traced.peak_allocation` | `MiB` |
| `process_rss` | `rss_before_inference_mb` | `process_rss.before` | `MiB` |
| `process_rss` | `rss_after_inference_mb` | `process_rss.after` | `MiB` |
| `process_rss` | `rss_delta_mb` | `process_rss.delta` | `MiB` |
| `pickle_size` | `serialized_model_size_mb` | `pickle.size` | `MiB` |
| `pickle_size` | `serialization_format` | `pickle.format` | `format` |

Custom measurement identifiers are derived from the strategy name and metric key, and their unit is `unspecified` unless users define their own interpretation outside Model Diet.

## Optional benchmark helper

The repeated demo benchmark is imported separately from `modeldiet.benchmark` and is provisional while the package is pre-1.0:

```python
from modeldiet.benchmark import run_repeated_benchmark
```

It trains built-in demonstration estimators on a bundled scikit-learn dataset. It is not a way to profile arbitrary user models; use `profile()` for that. The former `modeldiet.profiling.benchmark` import path and `modeldiet.profiling.isolated_runner` CLI remain available as compatibility shims. Installation extras are described in the [application guide](application.md#optional-benchmark-runner).

## Migrating from legacy benchmark model wrappers

The legacy `Model` wrapper and estimator-specific subclasses store mutable measurement values. Use an ordinary fitted estimator and the core result object instead:

```python
from sklearn.ensemble import RandomForestClassifier

from modeldiet import profile
from modeldiet.profiling import (
    LatencyMeasurement,
    ProcessRSSMeasurement,
    SerializationMeasurement,
    TracedAllocationMeasurement,
)

estimator = RandomForestClassifier(random_state=7).fit(X_train, y_train)
result = profile(
    estimator,
    X_test,
    measurements=[
        LatencyMeasurement(),
        TracedAllocationMeasurement(),
        ProcessRSSMeasurement(),
        SerializationMeasurement(),
    ],
)
mean_ms = result.measurements["latency"].metrics["mean_ms"].value
```

Use `run_repeated_benchmark()` for the bundled demonstration workflow and its summary output. `Report` is deprecated; the benchmark worker already prints its report and writes JSON. Replace `driver.main()` with explicit `run_repeated_benchmark(model_name=..., repetitions=...)` calls for the models you want to run.
