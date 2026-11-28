# Security Policy

## Supported Versions

Only the latest minor release receives security fixes. Older tags stay available
so pinned callers keep working, but they are not maintained.

| Version | Supported |
|---|---|
| `1.2.x` | yes |
| everything older | no |

## Reporting a Vulnerability

Use **[GitHub Security Advisories](https://github.com/studio-prisma/github-config-security-workflow/security/advisories/new)** — private by default. Do not open a public issue for a suspected vulnerability.

Include:

- affected version (tag or commit)
- what an attacker gains in a calling repository, not only what misbehaves
- reproduction steps, ideally with a minimal calling workflow

Expected response: an acknowledgement within 7 days, an assessment within 30.
This is a spare-time project, not a vendor with an on-call rotation — that is the
honest number, not a target.

## In Scope

- command injection into a run step through an input or an event field
- a download path that skips or weakens the checksum verification
- the workflow requesting or exposing more than `contents: read`
- a scanner silently not running while the job reports success
- the identity job printing matched text, or the pattern itself, to the log
- a way for a pull request to make a blocking scanner pass without a visible
  change to an ignore file in its diff

## Out of Scope

Documented design decisions are not findings:

- false positives or false negatives of the scanners themselves — report those
  upstream
- existing Semgrep findings not blocking (only new findings block, by design)
- Trivy ignoring HIGH/CRITICAL issues without an available fix (by design)
- checksum files coming from the same release as the binary (documented limit)
- Semgrep dependencies installed via pip without hash pinning (documented limit)
- anything requiring write access to the calling repository

## Handling of Secrets

The workflow accepts one secret, the identity pattern, and uses it in a single
step without printing it. A secret that reached a commit
is rotated first and removed second. The history stays readable either way.

## Disclosure

Coordinated. A fix ships first, the advisory follows once a patched release is
available. Reporters are credited unless they ask not to be.
