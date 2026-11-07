# Contributing to Model Diet

Thanks for helping improve Model Diet. Contributions should make profiling a fitted model and interpreting its inference measurements clearer and more dependable.

## Development setup

Model Diet requires Python 3.11 or later. CI tests Python 3.11, 3.12, 3.13, and 3.14. Create a virtual environment and install the development, test, and demo extras:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[test,demo,dev]"
python -m pip install -r requirements-docs.txt
```

In Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

The extras have separate purposes:

- The core package uses NumPy and psutil.
- `test` installs pytest, coverage support, pandas, and build tooling.
- `demo` installs the scikit-learn, pandas, and XGBoost example dependencies.
- `benchmark` and `benchmark-xgboost` install the dependencies for their respective benchmark workflows.
- `dev` installs Ruff, mypy, pip-audit, pre-commit, and supporting type stubs and distribution checks.
- `requirements-docs.txt` installs MkDocs Material.

## Quality checks

Run the full test suite and coverage threshold before opening a pull request:

```bash
python -m pytest -v
python -m pytest -v --cov=modeldiet --cov-branch --cov-fail-under=80 --cov-report=term-missing
```

Lint and format Python code with Ruff, then check production modules with the configured mypy baseline:

```bash
ruff check .
ruff format --check .
mypy
```

Install and run the local hooks before submitting:

```bash
python -m pip install pre-commit
pre-commit install
pre-commit run --all-files
```

The hooks check whitespace, end-of-file consistency, YAML/TOML syntax, large files, merge markers, Bandit findings, Ruff lint/format, and production-source typing. Dependency vulnerability audits run in CI rather than on every local commit:

```bash
python -m pip install --upgrade pip
pip-audit .
pip-audit --local --skip-editable
```

The local audit skips this repository's editable installation; it still audits installed non-editable dependencies. `pip-audit .` separately checks the current project requirements against the vulnerability database.

Validate the package and documentation when their respective files change:

```bash
python -m build
python -m twine check dist/*
python -m mkdocs build --strict
```

## Contribution workflow

1. Open an issue first for broad changes or new measurement areas so the expected behavior can be discussed.
2. Create a focused branch. Use descriptive prefixes such as `feat/`, `fix/`, `docs/`, `test/`, or `chore/`.
3. Keep each pull request focused. Include tests for behavior changes and update documentation when the API, measurements, dependencies, or user workflow changes.
4. Add user-visible changes to the `Unreleased` section of `CHANGELOG.md`. Tooling-only changes that do not affect package users do not need a changelog entry.
5. Report the checks you ran in the pull request. Resolve review comments and ensure the required CI jobs pass before merge.

## Testing conventions

- Use unit tests for validation rules, serialization, result construction, and isolated strategy behavior.
- Use integration tests when a workflow depends on a real estimator, package installation, subprocess, or file export. Keep generated files in temporary directories.
- Keep benchmark assertions about schema, metric presence, and valid ranges. Do not assert exact latency or memory numbers.
- Make benchmark inputs deterministic with fixed random seeds. Avoid mocks for workflows intended to exercise real training, measurement, or subprocess behavior.
- Test optional-dependency isolation in a subprocess or otherwise controlled import environment. Do not rely on `sys.modules` state left by other tests.
- Add tests for supported compatibility import paths when changing or deprecating a public interface.

## Review

Maintainers review pull requests for correctness, public API compatibility, measurement semantics, test coverage, documentation, and CI results. Keep the pull request description explicit about user-visible behavior, limitations, and any security or dependency changes.
