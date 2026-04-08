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

- `tsdl-version` input to pin the tsdl binary version (default: v2.0.0)
- `grammar-*-ref` inputs to pin each grammar to a specific version/tag/SHA
  - `grammar-bash-ref` (default: v0.25.1)
  - `grammar-json-ref` (default: v0.24.8)
  - `grammar-toml-ref` (default: v0.7.0)
  - `grammar-rbs-ref` (default: v0.2.2)
- `tsdl-version` output reporting the installed tsdl version

### Changed

- **BREAKING**: Grammar build now uses [tsdl](https://github.com/stackmystack/tsdl) instead of manual curl/unzip/gcc
  - Removes dependency on GCC being available in the runner
  - Grammars are built from pinned versions by default (previously used HEAD/master)
  - Grammars hosted outside `tree-sitter/` org (toml, rbs) are handled via tsdl's `from` override
- macOS support via tsdl's cross-platform binary releases

### Removed

- Manual grammar download, compile, and link steps (replaced by tsdl)
- Verbose diagnostic output in grammar installation step (tsdl provides its own progress)
- `grammar-jsonc` and `grammar-jsonc-ref` inputs — the standalone tree-sitter-jsonc grammar
  (WhyNotHugo/tree-sitter-jsonc) is archived and deprecated. JSONC is now supported by the
  main tree-sitter-json grammar (v0.24.0+). Use `grammar-json: true` instead.

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
