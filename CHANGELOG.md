# Changelog

All notable changes to this project will be documented in this file. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Python package
versions follow a four-component scheme `MAJOR.MINOR.PATCH.PACKAGE-REV[pre]` where
the first three components mirror the upstream `taskchampion` Rust crate.

## [Unreleased]

## [3.0.1.2] - 2026-05-22

First stable release of the fork. Pins TaskChampion Rust crate `3.0.1` exactly;
the fork follows the upstream crate version, with the fourth component tracking
package-only build / packaging revisions. `Development Status` promoted to
`5 - Production/Stable`.

### Added
- CI: new `test` job runs `pytest` against built Linux x86_64 wheels on Python 3.9–3.13;
  the `release` job now requires `test` to pass.
- PyPI trove classifiers: Python versions 3.9–3.13, MIT License, Development Status,
  Operating System, and Topic.
- README: dedicated `Installation` section with `pip install --pre taskchampion3-py-dev`,
  clarification of the PyPI-distribution-name vs Python-module-name distinction, and
  the list of pre-built wheel platforms.
- Test `test_set_status_unknown_raises` covering the new `Status.Unknown` write rejection.
- Tests covering previously-unverified Replica APIs: `test_undo` (full
  `get_undo_operations` + `commit_reversed_operations` round-trip), `test_expire_tasks`,
  `test_rebuild_working_set`, `test_dependency_map_cached` (`force=False` path).
- Smoke tests for sync error paths: `test_sync_to_remote_invalid_uuid` (verifies
  `ValueError` on malformed client_id, no network call) and
  `test_sync_to_gcp_invalid_credentials` (verifies `RuntimeError` on non-existent
  credential file path).
- `.cargo/audit.toml` with documented ignores for advisories that require an
  upstream `taskchampion` dependency bump to fix (idna 0.5.0, rustls-webpki
  0.102.x, adler unmaintained, rustls-pemfile unmaintained). The remaining
  RustSec advisories on transitive deps are expected to be cleared by running
  `cargo update` to bump within-semver-compatible versions (bytes, ring, time,
  tokio, futures-util, quinn-proto, rand).
- `Annotation` is now hashable (consistent with `Tag`) — instances can be used as
  `set()` elements or `dict` keys.
- `Operations` implements `Default` (Rust-side; no Python-visible change).
- CI: new `audit` job (cargo audit via `rustsec/audit-check@v2.0.0`) runs on push/PR
  and weekly cron (Monday 12h UTC) to catch RustSec advisories on Cargo dependencies.
  The job declares the `checks: write` and `issues: write` permissions required by
  `rustsec/audit-check` to post check annotations and open advisory issues.
- CI: new `mypy` job type-checks `taskchampion.pyi` and `tests/` on every push/PR.
- `taskchampion.pyi`: `Operations` now declares `__iter__` (previously missing despite
  the underlying `#[pyclass(sequence)]` making instances iterable at runtime).
- `.pre-commit-config.yaml` with hooks for ruff, ruff-format, cargo fmt,
  cargo clippy, and the standard whitespace/EOL checks. Install with
  `pre-commit install`.
- `[tool.ruff]` section in `pyproject.toml` (pycodestyle E/W, pyflakes F, isort I,
  flake8-bugbear B, pyupgrade UP). `tests/*` ignores B018 (false positive on
  `pytest.raises` blocks inspecting bare expressions).
- `CONTRIBUTING.md` documenting local setup, lint/test commands, and PR conventions.
- `SECURITY.md` documenting the vulnerability reporting channel and the weekly
  `cargo audit` policy.

### Changed
- `chrono` Cargo dependency bounded to `0.4` (was the unbounded `*`).
- `taskchampion.pyi`: stripped unneeded forward-reference quoting (e.g. `"Task"` →
  `Task`) since `.pyi` evaluation is always deferred; ruff `UP037` cleanup applied
  across the stub.
- Test imports sorted by ruff isort (8 files).
- `mypy.ini`: the `disable_error_code = assignment` suppression is now scoped to
  `[mypy-tests.*]` instead of being global, restoring strict type-checking on
  production code.
- `Operations.__getitem__` now accepts negative indices and slices. `ops[-1]` returns
  the last operation; `ops[a:b:s]` returns an `Operations`. Previously `ops[-1]`
  raised `OverflowError`.
- `Task.__repr__` and `TaskData.__repr__` now produce Python-idiomatic output
  (`Task(uuid=..., description=...)`, `TaskData(uuid=...)`) instead of leaking the
  Rust `Debug` formatter.
- `Operation.__repr__`, `Operations.__repr__`, `Annotation.__repr__`, `Tag.__repr__`,
  and `WorkingSet.__repr__` now produce Python-idiomatic output. Examples:
  `Operation.Create(uuid="...")`, `Operation.UndoPoint()`,
  `Operations([Operation.Update(...), ...])`,
  `Annotation(entry="2024-05-07T01:35:57+00:00", description="...")`,
  `Tag("user_tag")`, `WorkingSet(len=3, largest_index=4)`. `DependencyMap.__repr__`
  intentionally retains the underlying Rust `Debug` (no public iteration API to
  enumerate edges).
- `WorkingSet` iteration is now lazy (yields entries on demand via `by_index`)
  instead of materialising a `Vec` of all entries up front. Iteration semantics
  are unchanged.
- Wheels are now built against the PyO3 stable ABI (`abi3-py39`). A single wheel
  per platform now covers Python 3.9 and every later 3.x release. Previously,
  macOS and Windows produced only `cp312` wheels.
- CI: `checks.yml` is now a reusable workflow (`workflow_call`) invoked from
  `ci.yml` as a job. The `release` job now `needs: checks`, so clippy / rustfmt /
  ruff (lint + format) / mypy / cargo audit all gate a PyPI publish (previously
  they could be skipped on tag-only pushes).

### Removed
- `Task.into_task_data()` — renamed to `Task.to_task_data()` to match Python naming
  conventions (the method does not consume the task; it produces a copy).
- `black` as formatter (CI and pre-commit). Replaced by `ruff format`, which
  implements the same style as black with a single tool. `[tool.black]` removed
  from `pyproject.toml`; the `Python Formatting (black)` CI job is replaced by
  `Python Lint & Format (ruff)` which runs both `ruff check` and `ruff format --check`.
- Workflow `.github/workflows/publish_released_by_ci.yml` deleted. It was
  double-broken (used the deprecated `download-artifact@v4 name:` syntax for a
  cross-workflow download, referenced a non-existent `head_tag` field) and
  redundant with the `release` job in `ci.yml`, which already handles tag-based
  publishing with proper attestations. Its trigger also fired on `feat/**`
  branches and any commit message starting with `chore` — risk of unintended
  PyPI publish on dev branches.

### Fixed
- `Task.set_status(Status.Unknown, ops)` now raises `ValueError` instead of silently
  writing the literal string `"unknown status"` to the underlying task. The original
  status string is not preserved on read, so writing `Status.Unknown` back was
  corrupting data. Use `Task.set_value("status", "<value>", ops)` to set a custom
  status.

## [3.0.1.2a1] - 2026-05-22

### Changed
- Bumped Python package version to `3.0.1.2a1` (Rust crate target unchanged: `3.0.1`).
- README: corrected package name reference.

### Fixed
- CI: Windows build, release job dependency wiring, miscellaneous workflow fixes.

## [3.0.1.1.dev1] - 2026-05-16

### Changed
- Development release: Python package version 3.0.1.1.dev1 targeting TaskChampion crate 3.0.1.
- Packaging/tooling migrated to use the 'uv' package and pyproject.toml updated.
- Migrated underlying TaskChampion Rust crate to 3.0.1.
- Python bindings package (taskchampion-py) bumped to 3.0.1.1.dev1.
- Switched packaging/build tooling to use the 'uv' package and updated pyproject.toml accordingly.

### Removed
- Replica: removed new_task(), delete_task(), add_undo_point().
- Task: removed get/set/remove_uda(), get/set/remove_legacy_uda(), delete(); replaced by get/set/remove_user_defined_attribute(s)().

### Fixed
- working_set::by_uuid: unwrap() → now raises PyValueError on invalid UUID.
- dependency_map::dependencies/dependents: same fix.
- taskchampion.pyi: typing fixes (commit_reversed_operations -> bool; get_undo_operations -> Operations; remove_annotation timestamp -> datetime; TaskData.update -> Optional[str]).

### Added
- New bindings exposed in stubs: Replica.pending_tasks(), Replica.pending_task_data(), Replica.get_task_operations(); Task user-defined attribute APIs; TaskData.properties(), TaskData.items().
- AccessMode exported in __all__.

### Publication
- pyproject metadata updated (name: taskchampion3-py-fork) and packaging adjusted.

### Tests & build
- Tests: 85 -> 91 (+6).
- cargo build --release: 0 warnings.

Notes
- Python package version: 3.0.1.1.dev1
- Rust crate version: 3.0.1
