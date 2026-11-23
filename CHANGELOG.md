# Changelog

All notable changes to this project are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versions follow [SemVer](https://semver.org/).

## [Unreleased]

## [1.0.1] - 2026-11-23

### Added

- Reusable workflow `security.yml` with gitleaks 8.30.1, TruffleHog 3.99.8,
  Trivy 0.76.1 (filesystem and optional image) and Semgrep 1.183.0.
- Semgrep blocks only findings that are new against the previous commit or the
  PR base.
- Self-test in `validate.yml`, release from the tag with a CHANGELOG version gate.

### Fixed

- The image scan uses `BUILD_CONTEXT`; `DOCKER_CONTEXT` collides with the docker
  CLI ("no builder . found").

[Unreleased]: https://github.com/studio-prisma/github-config-security-workflow/compare/v1.0.1...HEAD
[1.0.1]: https://github.com/studio-prisma/github-config-security-workflow/releases/tag/v1.0.1
