# github-config-security-workflow

[Deutsche Fassung](README.de.md)

A reusable GitHub Actions workflow that runs an identity gate and four security
scanners and makes each of them a blocking check:

| Check | Fails on | Exceptions |
|---|---|---|
| Identity | private names (from a secret pattern) or public/private IPv4 addresses in any tracked file | IPv4 only: `.identityignore` |
| [gitleaks](https://github.com/gitleaks/gitleaks) | any finding in the full git history | `.gitleaks.toml`, `.gitleaksignore` |
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | keys that are verified live as still valid | none: rotate the key |
| [Trivy](https://github.com/aquasecurity/trivy) | HIGH/CRITICAL with a fix available (dependencies, misconfiguration, optional image) | `.trivyignore.yaml` with `statement` and `expired_at` |
| [Semgrep](https://github.com/semgrep/semgrep) | **new** findings against the previous commit or the PR base | `# nosemgrep: <rule-id>` with a reason |

## Requirements

- GitHub Actions on `ubuntu-latest`
- a `validate.yml` with an aggregating gate job (recommended)
- a secret `IDENTITY_PATTERN` (repository or organization) for the identity gate

## Usage

Add a job to the calling repository's workflow and list it in the gate's `needs`:

```yaml
jobs:
  security:
    uses: studio-prisma/github-config-security-workflow/.github/workflows/security.yml@<commit-sha> # vX.Y.Z
    with:
      semgrep-rules: "p/php p/security-audit"   # Python: "p/python p/security-audit"
      dockerfile: Dockerfile                    # omit = no image scan
    secrets:
      identity-pattern: ${{ secrets.IDENTITY_PATTERN }}
```

Always pin to the **commit** of a release tag, never to a branch. Dependabot
(`github-actions` ecosystem) proposes updates.

### Inputs

| Input | Default | Meaning |
|---|---|---|
| `semgrep-rules` | `p/security-audit` | Semgrep rule packs, space separated |
| `dockerfile` | empty | Dockerfile to build and scan; empty skips the image scan |
| `docker-context` | `.` | build context for the image scan |

| Secret | Meaning |
|---|---|
| `identity-pattern` | extended regex of names that must never appear (case-insensitive). Missing = the identity job fails, except on pull requests from forks, which receive no secrets |

## Identity gate

The pattern lives in a secret so that the list of names is not itself published.
On a hit the job prints `file:line` only, never the matching text, because the
log of a public repository is public too.

IPv4 addresses are allowed only from the documentation ranges (RFC 5737:
`192.0.2.x`, `198.51.100.x`, `203.0.113.x`), loopback, `0.0.0.0` and netmasks.
Private ranges count as findings, they reveal internal networks. Files that
contain four-part version numbers or example fixtures can be listed in
`.identityignore` (one glob per line); that file exempts them from the IPv4 check
only. The name check has no exceptions.

## Behaviour without a baseline

Scheduled runs, manual runs and the first push of a branch have nothing to
compare against. Semgrep then scans everything and only reports; the other
checks block as usual. Existing Semgrep findings therefore never block a change,
new ones always do.

## Workflow lint

A composite action for the lint job, pinned to the commit of a release tag as
well:

```yaml
      - name: actionlint
        uses: studio-prisma/github-config-security-workflow/.github/actions/actionlint@<commit-sha> # vX.Y.Z
```

It downloads actionlint from the official release and verifies the checksum.

## Supply chain

- scanner versions are pinned and at least two weeks old when adopted
- binaries are verified against the checksum file of their release
- no third-party actions; only `actions/checkout` and `actions/setup-python`
- the workflow requests `contents: read`; the only secret it accepts is the
  identity pattern, used in one step

Known limits: the checksum file comes from the same release as the binary, so it
protects against corrupted downloads, not against a compromised release.
Semgrep is installed via pip without hash-pinned dependencies. An outage of the
Semgrep rule registry fails the Semgrep job.

## License

[MIT](LICENSE)
