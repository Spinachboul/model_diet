# Changelog

This changelog records user-visible changes to Model Diet. Version 0.1.0 was published on production PyPI on November 7, 2026.

## [Unreleased]

## [0.1.0] - 2026-11-07

Initial production release.

### Added

- `modeldiet.profile()` for profiling a fitted model that implements `predict(X)`.
- Single-row inference latency by default, with percentile metrics and explicit metric definitions.
- Optional measurements for Python-traced allocations, process RSS snapshots, and pickle representation size.
- Structured `ProfileResult` objects with JSON and Markdown export.
- Optional runtime environment metadata and a configurable error-recording mode.
- Python package metadata and CI coverage for the supported Python versions.

### Changed

- The benchmark worker now trains ordinary scikit-learn/XGBoost estimators and profiles them through the public `profile()` API. Benchmark JSON field names and metric values retain their existing meanings.
- Historical benchmark JSON records were moved from the installable package tree to `benchmarks/artifacts/legacy-results/`; their recorded values are unchanged.
- The empty internal `modeldiet.profiling.complexity` module and empty `modeldiet.results` package were removed; neither provided symbols or was used internally.

### Deprecated

- `modeldiet.model.Model`, the estimator-specific model wrappers, `modeldiet.report.Report`, and `modeldiet.driver.main()` emit `DeprecationWarning` when instantiated or called. Their import paths remain available through the 0.x series and will not be removed before 1.0.0. See the API reference for migration guidance.
