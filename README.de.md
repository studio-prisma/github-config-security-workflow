# github-config-security-workflow

[English version](README.md)

Ein wiederverwendbarer GitHub-Actions-Workflow, der ein Identitäts-Gate und vier
Security-Scanner ausführt und jeden davon zum blockierenden Check macht:

| Check | Bricht ab bei | Ausnahmen |
|---|---|---|
| Identität | privaten Namen (aus einem Secret-Muster) oder öffentlichen/privaten IPv4-Adressen in einer versionierten Datei | nur IPv4: `.identityignore` |
| [gitleaks](https://github.com/gitleaks/gitleaks) | jedem Fund in der gesamten Git-History | `.gitleaks.toml`, `.gitleaksignore` |
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | Schlüsseln, die live als noch gültig geprüft wurden | keine: Schlüssel rotieren |
| [Trivy](https://github.com/aquasecurity/trivy) | HIGH/CRITICAL mit verfügbarem Fix (Abhängigkeiten, Fehlkonfiguration, optional Image) | `.trivyignore.yaml` mit `statement` und `expired_at` |
| [Semgrep](https://github.com/semgrep/semgrep) | **neuen** Funden gegenüber dem Vorgänger-Commit bzw. der PR-Basis | `# nosemgrep: <regel-id>` mit Begründung |

## Voraussetzungen

- GitHub Actions auf `ubuntu-latest`
- eine `validate.yml` mit sammelndem Gate-Job (empfohlen)
- ein Secret `IDENTITY_PATTERN` (Repo oder Organisation) für das Identitäts-Gate

## Verwendung

Im Workflow des aufrufenden Repos einen Job ergänzen und in die `needs` des Gates eintragen:

```yaml
jobs:
  security:
    uses: studio-prisma/github-config-security-workflow/.github/workflows/security.yml@<commit-sha> # vX.Y.Z
    with:
      semgrep-rules: "p/php p/security-audit"   # Python: "p/python p/security-audit"
      dockerfile: Dockerfile                    # weglassen = kein Image-Scan
    secrets:
      identity-pattern: ${{ secrets.IDENTITY_PATTERN }}
```

Immer auf den **Commit** eines Release-Tags pinnen, nie auf einen Branch.
Dependabot (Ökosystem `github-actions`) schlägt Updates vor.

### Eingaben

| Eingabe | Standard | Bedeutung |
|---|---|---|
| `semgrep-rules` | `p/security-audit` | Semgrep-Regelpakete, durch Leerzeichen getrennt |
| `dockerfile` | leer | Dockerfile, das gebaut und gescannt wird; leer = kein Image-Scan |
| `docker-context` | `.` | Build-Kontext für den Image-Scan |

| Secret | Bedeutung |
|---|---|
| `identity-pattern` | erweiterter Regex der Namen, die nie vorkommen dürfen (ohne Groß-/Kleinschreibung). Fehlt es, scheitert der Identitäts-Job, außer bei Pull Requests aus Forks, die keine Secrets bekommen |

## Identitäts-Gate

Das Muster liegt in einem Secret, damit die Namensliste nicht selbst veröffentlicht
wird. Bei einem Treffer gibt der Job nur `Datei:Zeile` aus, nie den gefundenen
Text, denn auch das Log eines öffentlichen Repos ist öffentlich.

IPv4-Adressen sind nur aus den Dokumentations-Bereichen (RFC 5737: `192.0.2.x`,
`198.51.100.x`, `203.0.113.x`), Loopback, `0.0.0.0` und Netzmasken erlaubt.
Private Bereiche zählen als Fund, sie verraten interne Netze. Dateien mit
vierstelligen Versionsnummern oder Beispiel-Fixtures können in `.identityignore`
stehen (ein Glob je Zeile); das nimmt sie nur von der IPv4-Prüfung aus. Die
Namensprüfung kennt keine Ausnahmen.

## Verhalten ohne Vergleichsstand

Zeitgesteuerte Läufe, manuelle Starts und der erste Push eines Branches haben
keinen Vergleichsstand. Semgrep scannt dann alles und berichtet nur; die
anderen Checks blockieren wie gewohnt. Bestehende Semgrep-Funde blockieren damit
nie eine Änderung, neue immer.

## Lieferkette

- Scanner-Versionen sind fest und bei Übernahme mindestens zwei Wochen alt
- Binaries werden gegen die Prüfsummen-Datei ihres Releases geprüft
- keine Actions von Dritten, nur `actions/checkout` und `actions/setup-python`
- der Workflow fordert `contents: read` an; das einzige Secret, das er annimmt, ist
  das Identitäts-Muster, genutzt in einem Schritt

Bekannte Grenzen: Die Prüfsummen-Datei stammt aus demselben Release wie das
Binary; sie schützt vor beschädigten Downloads, nicht vor einem kompromittierten
Release. Semgrep wird per pip ohne hash-gepinnte Abhängigkeiten installiert.
Ein Ausfall der Semgrep-Regel-Registry lässt den Semgrep-Job scheitern.

## Lizenz

[MIT](LICENSE)
