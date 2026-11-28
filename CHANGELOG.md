# Changelog

All notable changes to this project are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versions follow [SemVer](https://semver.org/).

## [Unreleased]

## [1.2.1] - 2026-11-28

### Fixed

- The actionlint composite action could not be used: `.gitattributes` excluded all of `.github/`
  from the source archive the runner downloads, so `action.yml` was missing ("Can't find
  'action.yml'"). Only workflows, issue templates, the PR template and the Dependabot config are
  excluded now.

## [1.2.0] - 2026-11-28

### Added

- Composite action `.github/actions/actionlint`: installs actionlint 1.7.12 straight from the
  rhysd/actionlint release and verifies it against the release checksums. Replaces
  `raven-actions/actionlint@v2`, a third-party action pinned by tag only.

### Changed

- All actions are pinned to commit SHAs instead of tags, version as comment. Dependabot keeps
  the pins current.

## [1.1.0] - 2026-11-23

### Added

- Identity gate: fails on private names from the `identity-pattern` secret and on
  IPv4 addresses outside the documentation ranges. Prints file:line only.
  `.identityignore` exempts files from the IPv4 check (not from the name check).

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

[Unreleased]: https://github.com/studio-prisma/github-config-security-workflow/compare/v1.2.1...HEAD
[1.2.1]: https://github.com/studio-prisma/github-config-security-workflow/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/studio-prisma/github-config-security-workflow/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/studio-prisma/github-config-security-workflow/compare/v1.0.1...v1.1.0
[1.0.1]: https://github.com/studio-prisma/github-config-security-workflow/releases/tag/v1.0.1
