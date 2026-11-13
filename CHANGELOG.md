# Changelog

All notable changes to this project are documented here. Versions follow [Semantic Versioning](https://semver.org/).

## Unreleased

### Fixed
- Range notation uses `-` or "부터…까지" instead of `~`, which GitHub renders as strikethrough; reports follow the same rule.

## 1.0.0 - 2026-11-13

### Added
- `location-privacy-kr` skill with four modes: `audit`, `fix`, `docs`, `prep`.
- Location data map from code for any stack (Android, iOS including Objective-C, React Native, Flutter, web, backend).
- Checklists for location-based service terms (T1-T22), privacy policy (P1-P20) and personal location information items (L1-L7), consent and rights (C, R, N, K), and safeguards (S1-S5).
- Fix patterns: consent enforcement, withdrawal and pause, automatic use/provision records, minimal collection, deletion, third-party notice, age gating, and document skeletons.
- 17 rulings on where the five source skills conflicted (`references/conflicts.md`).
