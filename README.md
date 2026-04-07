<p align="center">
  <a href="https://custodia.dev">
    <img src="https://custodia.dev/custodia_shield_transparent.png" alt="Custodia Security Scan" width="120" />
  </a>
</p>

# Custodia Security Scan

### Ship code. Not vulnerabilities.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Custodia%20Security%20Scan-blue?logo=github&logoColor=white)](https://github.com/marketplace/actions/custodia-security-scan)
[![Latest Release](https://img.shields.io/github/v/release/contactdavidpersonal-code/custodia-scan-action?label=version&color=00e5ff)](https://github.com/contactdavidpersonal-code/custodia-scan-action/releases)
[![Free Tier Available](https://img.shields.io/badge/free%20tier-included-brightgreen)](https://custodia.dev)
[![Custodia Score: 99/100](https://custodia.dev/api/reports/a080d199-ba29-462b-b855-87d409d519dc/badge)](https://custodia.dev/reports/a080d199-ba29-462b-b855-87d409d519dc)

OWASP Top 10 + AI security scanning on every push and pull request — with compliance mapping to SOC 2, NIST CSF, EU AI Act, and ISO 42001. Two-line setup. No config required.

---

---

## Why Custodia

Most teams only do security reviews when something breaks. Custodia makes security a first-class part of your CI pipeline — every commit, every PR, automatically.

- **Catches what linters miss** — auth flaws, injection vectors, hardcoded secrets, insecure dependencies
- **Not just rules — AI reasoning** — a 5-stage pipeline verifies every CRITICAL finding before flagging it, keeping false positive rates low
- **PR-ready diff mode** — only changed files are scanned on pull requests; runs in seconds
- **Compliance out of the box** — findings map to SOC 2 controls, CWE IDs, and OWASP categories automatically
- **No per-seat pricing** — one subscription, the whole team

---

## Quick Setup (2 steps)

### Step 1 — Add your API key as a secret

Get a free key at **[custodia.dev](https://custodia.dev)** (no card required). Then in your repo:

**Settings → Secrets and variables → Actions → New repository secret**

| Name | Value |
|---|---|
| `CUSTODIA_API_KEY` | your key from the dashboard |

### Step 2 — Add the workflow

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
          fetch-depth: 0   # required for diff mode on PRs

      - uses: contactdavidpersonal-code/custodia-scan-action@v1
        with:
          api-key: ${{ secrets.CUSTODIA_API_KEY }}
```

Done. Every push to `main` gets a full scan. Every PR gets a fast diff scan on only the changed files.

---

## What the Output Looks Like

Findings appear as native **GitHub Actions annotations** — directly inline on the PR diff:

```
::error file=src/config/db.ts::  [CRITICAL] SEC-01 — Hardcoded database password  (CWE-798 · SOC2 CC6.1)
::error file=src/routes/admin.ts::  [HIGH] AUTH-02 — Missing authentication on /admin/*  (CWE-306 · OWASP A01)
::warning file=src/middleware/::  [MEDIUM] LOG-01 — No security event logging  (SOC2 CC7.2)
```

Plus a full terminal summary:

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

## Configuration

### Inputs

| Input | Description | Default |
|---|---|---|
| `api-key` | **Required.** Your Custodia API key | — |
| `mode` | `auto` — diff on PRs, full on push · `diff` — always diff · `full` — always full | `auto` |
| `path` | Directory to scan | `.` |
| `fail-on` | Fail the build if findings exist at this severity or above: `CRITICAL` · `HIGH` · `MEDIUM` · `LOW` · `none` | `HIGH` |
| `args` | Extra arguments passed directly to `custodia scan` | `''` |

### Outputs

| Output | Description |
|---|---|
| `score` | Security score (0–100) |
| `findings-count` | Total number of findings |
| `report-path` | Path to the generated Markdown report artifact |

---

## Examples

### Only block on CRITICAL findings

```yaml
- uses: contactdavidpersonal-code/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    fail-on: CRITICAL
```

### Scan a subdirectory (monorepos)

```yaml
- uses: contactdavidpersonal-code/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    path: ./backend
```

### Use the score as a gate in a downstream step

```yaml
- uses: contactdavidpersonal-code/custodia-scan-action@v1
  id: scan
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}

- name: Enforce score threshold
  run: |
    if [ "${{ steps.scan.outputs.score }}" -lt "70" ]; then
      echo "Security score below 70 — blocking merge"
      exit 1
    fi
```

### Non-blocking scan (report only, never fail the build)

```yaml
- uses: contactdavidpersonal-code/custodia-scan-action@v1
  with:
    api-key: ${{ secrets.CUSTODIA_API_KEY }}
    fail-on: none
```

---

## Coverage

| Category | What's scanned |
|---|---|
| **OWASP Top 10** | A01 Broken Access Control, A02 Crypto Failures, A03 Injection, A04 Insecure Design, A05–A10 |
| **OWASP LLM Top 10** | Prompt injection, insecure output handling, excessive agency, model DoS, supply chain |
| **Dependency CVEs** | OSV.dev-backed; npm, pip, gem, cargo, go.mod |
| **Secrets** | Hardcoded keys, tokens, passwords, connection strings |
| **SOC 2 TSC** | CC1–CC9 Common Criteria mapping (Dev+ plans) |
| **NIST CSF** | Identify · Protect · Detect · Respond (Dev+ plans) |
| **EU AI Act** | Art. 9, 14, 52 (Pro+ plans) |
| **ISO 42001** | A.6–A.10 AI management (Pro+ plans) |
| **CWE Top 25** | All scans |

---

## Pricing

No per-seat pricing. One subscription covers everyone on your team.

| Plan | Price | Full Scans / mo | Diff Scans / mo |
|---|---|---|---|
| **Free** | $0 | 3 | 5 |
| **Dev** | $39 | 10 | 60 |
| **Pro** | $89 | 25 | 150 |
| **Business** | $249 | 60 | 400 |

**Diff scans are built for CI/CD.** Only changed files are sent, they complete in seconds, and they count against the lower diff quota. A team running 10–15 PRs/week easily stays on the free tier.

[See full pricing →](https://custodia.dev/billing)

---

## FAQ

**Does this work with private repos?**
The CLI reads files in the checked-out workspace and sends them to the Custodia API for analysis. No code is stored permanently.

**How many scans will a busy CI pipeline use?**
PRs use diff scans (quota-efficient). A team with 50 pull requests per month comfortably runs on the Dev plan (60 diff scans/mo).

**Does it support monorepos?**
Yes — use the `path` input to point at a subdirectory. Run multiple action steps for different services.

**Which languages are supported?**
JavaScript/TypeScript, Python, Ruby, Go, Rust, Java, PHP, and more. Language detection is automatic.

**What about false positives?**
The pipeline includes a dedicated CRITICAL verifier stage that re-checks every critical finding and downgrades unconfirmed ones before the report is returned. False positive rate is significantly lower than rule-based scanners.

**Can I use this with a self-hosted runner?**
Yes — as long as the runner has internet access and Node.js ≥18.

---

## Links

- [Get a free API key](https://custodia.dev/dashboard)
- [CLI reference & docs](https://custodia.dev/help)
- [Pricing](https://custodia.dev/billing)
- [Blog](https://custodia.dev/blog)
- [custodia.dev](https://custodia.dev)
