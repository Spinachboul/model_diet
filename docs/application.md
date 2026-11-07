# Application Guide

This guide explains how to use Model Diet in a Python application. Model Diet profiles an already-fitted model; it does not train a model or evaluate its predictive quality.

See the [API reference](api.md) for supported import paths and compatibility expectations.

## Installation

Model Diet requires Python 3.11 or later. Install the core package from production PyPI:

```bash
python -m pip install modeldiet
```

Install the optional demo dependencies to use the scikit-learn examples in this guide:

```bash
python -m pip install "modeldiet[demo]"
```

## Optional benchmark runner

The benchmark runner loads the bundled dataset as a pandas DataFrame. Install `modeldiet[benchmark]` for its Logistic Regression or Random Forest workflows; `modeldiet[benchmark-xgboost]` adds XGBoost. Both extras include pandas. The `modeldiet[demo]` extra includes pandas, scikit-learn, and XGBoost.

```python
from modeldiet.benchmark import run_repeated_benchmark

run_repeated_benchmark("lr", repetitions=1, output_dir="benchmark-output")
```

The core `modeldiet` installation does not include scikit-learn or XGBoost. Profiling your own model with `profile()` requires neither library.
The old `modeldiet.profiling.benchmark` import path remains available as a compatibility shim.
For the legacy model wrappers, `Report`, and `driver.main()` deprecation details and migration examples, see the [API compatibility policy](api.md#api-stability-and-support-tiers).

## Profile a fitted model

Any fitted Python object with a callable `predict(X)` method can be profiled, provided it accepts the input rows described below. This example creates a classifier, fits it, and profiles its latency:

```python
import numpy as np
from sklearn.ensemble import RandomForestClassifier

from modeldiet import profile

rng = np.random.default_rng(7)
X = rng.random((100, 5))
y = rng.integers(0, 2, size=100)

model = RandomForestClassifier(n_estimators=20, random_state=7)
model.fit(X, y)

result = profile(model, X)
mean_ms = result.measurements["latency"].metrics["mean_ms"].value
print(f"Mean single-row latency: {mean_ms:.3f} ms")
```

`X` should represent the data and preprocessing used at inference time. It must be non-empty, two-dimensional tabular data supplied as a NumPy array, pandas DataFrame, or list or tuple of rows. Model Diet preserves row batches of size one when calling `predict()`.

## Default latency profile

If `measurements` is omitted, `profile()` runs `LatencyMeasurement`. It makes one warmup prediction by default, then times each row separately with `time.perf_counter()`. The resulting `latency` measurement includes the mean, p50, p90, p95, and p99 latency in milliseconds, plus the number of timed rows.

This is inference latency only. Model Diet does not compare predictions against labels, so it does not calculate accuracy or other quality metrics.

## Additional measurements

Built-in strategies can be selected from `modeldiet.profiling`:

```python
from modeldiet import profile
from modeldiet.profiling import (
    LatencyMeasurement,
    ProcessRSSMeasurement,
    SerializationMeasurement,
    TracedAllocationMeasurement,
)

result = profile(
    model,
    X,
    measurements=[
        LatencyMeasurement(warmup_iterations=2),
        TracedAllocationMeasurement(),
        ProcessRSSMeasurement(),
        SerializationMeasurement(),
    ],
    include_environment=True,
)
```

Supplying `measurements` replaces the default strategy list. Include `LatencyMeasurement()` explicitly if latency should be measured alongside other strategies.

| Strategy | What it reports | Important interpretation |
| --- | --- | --- |
| `LatencyMeasurement` | Per-row mean and percentile latency | Timed without `tracemalloc`; one prediction per row, after warmup. |
| `TracedAllocationMeasurement` | Peak Python-traced allocation during a prediction pass | Does not include native memory allocations that `tracemalloc` cannot track. |
| `ProcessRSSMeasurement` | Process RSS before and after inference and their difference | Whole-process endpoint snapshots, not model-only memory or peak RSS. |
| `SerializationMeasurement` | Size of `pickle.dumps(model)` and format | Describes the pickle representation; it does not make pickle portable or safe to load. |

Each strategy runs in a separate pass over the data. The extra passes can affect total runtime, and the measurements are not simultaneous observations.

## Read and export results

The result is a `ProfileResult`. Its `measurements` mapping is keyed by strategy name. Successful entries contain `MetricValue` objects; access the numeric value and its definition like this:

```python
measurement = result.measurements["latency"]
mean_metric = measurement.metrics["mean_ms"]
print(mean_metric.value)
print(mean_metric.definition.unit)
print(mean_metric.definition.semantics)
```

The result also records the schema version, model class and module, input type, sample and feature counts, and optionally environment metadata. `include_environment=True` records Python, operating system, CPU and memory information, selected package versions, and common thread environment variables.

Export the same result to JSON or Markdown:

```python
result.to_json("artifacts/profile.json")
result.to_markdown("artifacts/profile.md")
```

Create the `artifacts` directory before running this example; the export methods do not create parent directories. JSON uses schema version 1, and non-finite floating-point values are serialized as `null`.

## Error handling and custom strategies

By default, a measurement error raises `RuntimeError` and stops profiling. Set `on_error="record"` to store the exception type and message in that measurement's result and continue with the remaining strategies:

```python
result = profile(model, X, measurements=[LatencyMeasurement()], on_error="record")
```

Custom strategies subclass `BaseMeasurement` and implement `measure(model, X=None)`, returning a mapping of serializable metric names and values. Give each strategy a unique `name`. Model Diet supplies generic definitions for custom metrics; it cannot determine the units or semantics of custom values, so document them in the strategy.

## Interpreting and comparing profiles

Latency depends on hardware, operating system, Python and model-library versions, background load, and model configuration. Use representative inputs and compare profiles under similar conditions. Percentiles describe the per-row timings in the supplied sample; they are not service-level guarantees.

RSS delta may be negative or near zero because it is the difference between two process-wide memory snapshots. It is not a peak measurement and may reflect unrelated allocations or memory reclamation. Python-traced allocation excludes native allocations. Pickle size is not the same as on-disk size for every model storage format.

## Current boundaries

- Inputs are tabular and non-empty; each prediction call receives one row at a time.
- The model must already be fitted and expose `predict()`.
- There are no built-in adapters for PyTorch, TensorFlow, or ONNX Runtime.
- GPU memory, batch throughput, concurrent request behavior, and peak RSS are not currently measured.
- Accuracy and other predictive quality metrics are outside the scope of `profile()`.
