# Changelog

All notable changes to this project will be documented in this file. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Python package
versions follow a four-component scheme `MAJOR.MINOR.PATCH.PACKAGE-REV[pre]` where
the first three components mirror the upstream `taskchampion` Rust crate.

## [Unreleased]

### Added
- CI: new `test` job runs `pytest` against built Linux x86_64 wheels on Python 3.9–3.13;
  the `release` job now requires `test` to pass.
- PyPI trove classifiers: Python versions 3.9–3.13, MIT License, Development Status,
  Operating System, and Topic.
- README: dedicated `Installation` section with `pip install --pre taskchampion3-py-dev`,
  clarification of the PyPI-distribution-name vs Python-module-name distinction, and
  the list of pre-built wheel platforms.
- Test `test_set_status_unknown_raises` covering the new `Status.Unknown` write rejection.
- `Annotation` is now hashable (consistent with `Tag`) — instances can be used as
  `set()` elements or `dict` keys.
- `Operations` implements `Default` (Rust-side; no Python-visible change).

### Removed
- `Task.into_task_data()` — renamed to `Task.to_task_data()` to match Python naming
  conventions (the method does not consume the task; it produces a copy).

### Changed
- `chrono` Cargo dependency bounded to `0.4` (was the unbounded `*`).
- Wheels are now built against the PyO3 stable ABI (`abi3-py39`). A single wheel
  per platform now covers Python 3.9 and every later 3.x release. Previously,
  macOS and Windows produced only `cp312` wheels.

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
