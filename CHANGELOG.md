# Changelog

All notable changes to this project will be documented in this file.

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
