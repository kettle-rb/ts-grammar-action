# Changelog

[![SemVer 2.0.0][📌semver-img]][📌semver] [![Keep-A-Changelog 1.0.0][📗keep-changelog-img]][📗keep-changelog]

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog][📗keep-changelog],
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html),
and [yes][📌major-versions-not-sacred], platform and engine support are part of the [public API][📌semver-breaking].
Please file a bug if you notice a violation of semantic versioning.

[📌semver]: https://semver.org/spec/v2.0.0.html
[📌semver-img]: https://img.shields.io/badge/semver-2.0.0-FFDD67.svg?style=flat
[📌semver-breaking]: https://github.com/semver/semver/issues/716#issuecomment-869336139
[📌major-versions-not-sacred]: https://tom.preston-werner.com/2022/05/23/major-version-numbers-are-not-sacred.html
[📗keep-changelog]: https://keepachangelog.com/en/1.0.0/
[📗keep-changelog-img]: https://img.shields.io/badge/keep--a--changelog-1.0.0-FFDD67.svg?style=flat

## [Unreleased]

### Added

- Enhanced diagnostic output in "Configure grammar paths" step
  - Shows current environment state and all input variables
  - Lists installed grammar files with full paths
  - Confirms each environment variable being set with visual indicators (✓, ℹ)
  - Reports when LD_LIBRARY_PATH already contains the library path
- New "Verify grammar environment" step runs after configuration
  - Verifies GITHUB_ENV updates persisted to new shell
  - Shows all grammar-related environment variables
  - Checks that grammar files exist and are readable
  - Validates exported symbols using `nm` command
  - Reports issues as warnings without failing the workflow

### Changed

### Deprecated

### Removed

### Fixed

### Security

## [1.0.0] - 2025-12-29

- TAG: [v1.0.0][1.0.0t]

### Added

- Diagnostic output when building grammars to help troubleshoot tree-sitter setup issues
  - Shows libtree-sitter installation location
  - Shows tree-sitter header availability
  - Shows LD_LIBRARY_PATH configuration
- Symbol verification after grammar compilation
  - Checks that `tree_sitter_<lang>` symbol is exported from built library
  - Warns if symbol is not found

### Changed

- Grammar compilation now includes `/usr/local/include` in include path
  - Ensures tree-sitter headers from `tree-sitter/setup-action` are found

[Unreleased]: https://github.com/kettle-rb/ts-grammar-action/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/kettle-rb/ts-grammar-action/compare/4adf5a6f4960c7e8d0fcc1d05165e3647eb5cc8c...v1.0.0
[1.0.0t]: https://github.com/kettle-rb/ts-grammar-action/tags/v1.0.0
