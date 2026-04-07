# Custodia Security Scan

[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Custodia%20Security%20Scan-blue?logo=github)](https://github.com/marketplace/actions/custodia-security-scan)
[![Custodia.dev Score: 99/100](https://custodia.dev/api/reports/a080d199-ba29-462b-b855-87d409d519dc/badge)](https://custodia.dev/reports/a080d199-ba29-462b-b855-87d409d519dc)

**OWASP Top 10 + AI security scanning on every push and pull request.** Maps findings to SOC 2, NIST CSF, EU AI Act, and ISO 42001. Zero config. Free tier included.

---

## What It Does

- **OWASP Top 10** — auth issues, injection flaws, XSS, hardcoded secrets, misconfigured components
- **AI Security (OWASP LLM Top 10)** — prompt injection, insecure output handling, excessive agency, model DoS
- **Dependency CVEs** — OSV.dev powered; finds vulnerable packages in npm, pip, gem, cargo, go
- **Compliance mapping** — flags map to SOC 2 CC controls, NIST CSF, CWE, EU AI Act articles
- **Diff mode** — on pull requests, only changed files are scanned (fast, quota-efficient)
- **Fails on severity threshold** — configurable; defaults to failing on HIGH or CRITICAL findings
- **Artifacts** — uploads `.custodia-reports/` as a workflow artifact on every run

---

## Quick Setup

### 1. Get a free API key

Sign up at **[custodia.dev](https://custodia.dev)** (free tier — 3 full scans + 5 diff scans/month, no card required).

Copy your API key from the dashboard under **Identity & Access → Generate Key**.

### 2. Add the secret

In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**

- Name: `CUSTODIA_API_KEY`
- Value: your API key

### 3. Add the workflow

Create `.github/workflows/custodia.yml`:

```yaml
name: Security Scan

on:
  push:
    branches: ["main", "master"]
  pull_request:

jobs:
  security:
    name: Custodia Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # required for diff mode

      - uses: custodia/custodia-scan-action@v1
        with:
          api-key: ${{ secrets.CUSTODIA_API_KEY }}
```

That's it. The action runs on every push to main and every pull request.

---

## Configuration

| Input | Description | Default |
|---|---|---|
| `api-key` | Custodia API key (required) | — |
| `mode` | `auto` — diff on PRs, full on push. `diff` — always diff. `full` — always full. | `auto` |
| `path` | Directory to scan | `.` |
| `fail-on` | Fail if findings at this severity exist: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `none` | `HIGH` |
| `args` | Extra arguments passed to `custodia scan` | `''` |

### Outputs

| Output | Description |
|---|---|
| `score` | Security score 0–100 |
| `findings-count` | Total findings |
| `report-path` | Path to generated Markdown report |

---

## Examples

### Only fail on CRITICAL findings

```yaml
- uses: custodia/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    fail-on: CRITICAL
```

### Always run a full scan (no diff)

```yaml
- uses: custodia/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    mode: full
```

### Scan a subdirectory

```yaml
- uses: custodia/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    path: ./backend
```

### Use the score in a downstream step

```yaml
- uses: custodia/custodia-scan-action@v1
  id: scan
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}

- name: Print score
  run: echo "Security score: ${{ steps.scan.outputs.score }}"
```

### Non-blocking scan (report but never fail the build)

```yaml
- uses: custodia/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    fail-on: none
  continue-on-error: true
```

---

## What the Output Looks Like

```
┌─────────────────────────────────────────────────────┐
│  CUSTODIA SECURITY SCAN                             │
│  Score: 78 / 100   ·   Verdict: NEEDS ATTENTION    │
└─────────────────────────────────────────────────────┘

  [CRITICAL]  SEC-01   src/config/db.ts        hardcoded database password
  [HIGH]      AUTH-02  src/routes/admin.ts     missing authentication on /admin/*
  [HIGH]      INJ-01   src/api/search.ts       SQL injection via unsanitized input
  [MEDIUM]    LOG-01   src/middleware/         no security event logging

  4 findings  ·  3 files  ·  1 critical

  SOC 2 CC6.1 · CC6.6 · CC7.2  ·  OWASP A02 · A07  ·  CWE-798 · CWE-89
```

---

## Pricing

| Plan | Price | Full Scans | Diff Scans |
|---|---|---|---|
| **Free** | $0/mo | 3/mo | 5/mo |
| **Dev** | $39/mo | 10/mo | 60/mo |
| **Pro** | $89/mo | 25/mo | 150/mo |
| **Business** | $249/mo | 60/mo | 400/mo |

No per-seat pricing. One subscription covers your entire team.

**Diff scans are the right mode for CI/CD** — only changed files are sent, they run in seconds, and they count against the lower quota. A team pushing 10–15 times per week easily stays on the free tier.

[See full pricing and features →](https://custodia.dev/billing)

---

## Compliance Coverage

| Framework | What's covered |
|---|---|
| **OWASP Top 10** | A01–A10 (all categories) |
| **OWASP LLM Top 10** | LLM01–LLM10 (AI/ML security) |
| **SOC 2 TSC** | CC1–CC9 (Common Criteria) — Dev+ |
| **NIST CSF** | Identify, Protect, Detect, Respond — Dev+ |
| **EU AI Act** | Art.9, Art.14, Art.52 — Pro+ |
| **ISO 42001** | A.6–A.10 AI management — Pro+ |
| **CWE Top 25** | All scans |

---

## FAQ

**Does this work with private repos?**  
Yes — the CLI only reads files in the checked-out workspace. No code is stored permanently.

**How many scans will a busy CI pipeline use?**  
Diff scans (the default on PRs) are quota-efficient. A team with 50 pull requests per month comfortably runs on the Dev plan (60 diff scans/mo).

**Does it support monorepos?**  
Yes — use the `path` input to point at a subdirectory. Run multiple instances of the action for different services.

**Which languages are supported?**  
JavaScript/TypeScript, Python, Ruby, Go, Rust, Java, PHP, and more. Language detection is automatic.

**What about false positives?**  
The 5-stage AI pipeline includes a dedicated critical verifier (Stage 2.5) that re-checks every CRITICAL finding and downgrades unconfirmed ones to HIGH. False positive rate is significantly lower than rule-based tools.

---

## Links

- [Dashboard & API keys](https://custodia.dev/dashboard)
- [Docs & CLI reference](https://custodia.dev/help)
- [Pricing](https://custodia.dev/billing)
- [Blog](https://custodia.dev/blog)
- [custodia.dev](https://custodia.dev)
