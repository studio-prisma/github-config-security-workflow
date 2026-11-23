# github-config-security-workflow

[Deutsche Fassung](README.de.md)

A reusable GitHub Actions workflow that runs four security scanners and makes
each of them a blocking check:

| Scanner | Fails on | Exceptions |
|---|---|---|
| [gitleaks](https://github.com/gitleaks/gitleaks) | any finding in the full git history | `.gitleaks.toml`, `.gitleaksignore` |
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | keys that are verified live as still valid | none: rotate the key |
| [Trivy](https://github.com/aquasecurity/trivy) | HIGH/CRITICAL with a fix available (dependencies, misconfiguration, optional image) | `.trivyignore.yaml` with `statement` and `expired_at` |
| [Semgrep](https://github.com/semgrep/semgrep) | **new** findings against the previous commit or the PR base | `# nosemgrep: <rule-id>` with a reason |

## Requirements

- GitHub Actions on `ubuntu-latest`
- a `validate.yml` with an aggregating gate job (recommended)

## Usage

Add a job to the calling repository's workflow and list it in the gate's `needs`:

```yaml
jobs:
  security:
    uses: studio-prisma/github-config-security-workflow/.github/workflows/security.yml@<commit-sha> # vX.Y.Z
    with:
      semgrep-rules: "p/php p/security-audit"   # Python: "p/python p/security-audit"
      dockerfile: Dockerfile                    # omit = no image scan
```

Always pin to the **commit** of a release tag, never to a branch. Dependabot
(`github-actions` ecosystem) proposes updates.

### Inputs

| Input | Default | Meaning |
|---|---|---|
| `semgrep-rules` | `p/security-audit` | Semgrep rule packs, space separated |
| `dockerfile` | empty | Dockerfile to build and scan; empty skips the image scan |
| `docker-context` | `.` | build context for the image scan |

## Behaviour without a baseline

Scheduled runs, manual runs and the first push of a branch have nothing to
compare against. Semgrep then scans everything and only reports; the other
three scanners block as usual. Existing Semgrep findings therefore never block
a change, new ones always do.

## Supply chain

- scanner versions are pinned and at least two weeks old when adopted
- binaries are verified against the checksum file of their release
- no third-party actions; only `actions/checkout` and `actions/setup-python`
- the workflow requests `contents: read` and receives no secrets

Known limits: the checksum file comes from the same release as the binary, so it
protects against corrupted downloads, not against a compromised release.
Semgrep is installed via pip without hash-pinned dependencies. An outage of the
Semgrep rule registry fails the Semgrep job.

## License

[MIT](LICENSE)
