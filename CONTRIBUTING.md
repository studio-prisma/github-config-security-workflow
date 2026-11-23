# Contributing

## Setup

Nothing to install for the workflow itself. For local checks you need
[actionlint](https://github.com/rhysd/actionlint) (with shellcheck on `PATH`
for script checks).

## Checks before a pull request

```bash
actionlint
```

The self-test in `validate.yml` runs the workflow against this repository on
every push and pull request. It must be green.

## Rules for changes

- **Scanner versions:** pin exact versions that are at least two weeks old.
  Update the version in `security.yml`, `README.md` and `CHANGELOG.md` together.
- **Downloads:** every binary is verified against the checksum file of its
  release. No `curl | sh`, no `latest`.
- **No third-party actions.** GitHub's own `actions/*` only.
- **Inputs never go into a script directly.** Pass them through `env:` and quote
  them in the shell.
- **Behaviour changes** (what blocks, what only reports) are a minor version at
  least; removing or renaming an input is a major version.

## Release

1. Rename `## [Unreleased]` to `## [X.Y.Z] - YYYY-MM-DD`, add a new empty
   `Unreleased` block and the compare links.
2. `git commit -m "chore(release): vX.Y.Z"`
3. `git tag -a vX.Y.Z -m "vX.Y.Z"` and push the tag.
4. `release.yml` checks the version against `CHANGELOG.md` and publishes the
   release notes.

Callers pin the commit of the tag; tags are protected and cannot be moved.

## Commits

Conventional Commits, imperative, English. No co-author trailers.
