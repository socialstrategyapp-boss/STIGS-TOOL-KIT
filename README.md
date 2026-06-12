# Stigs Tool Kit

A defense-in-depth security toolkit for repository, CI/CD, and supply-chain hardening.
All workflows run out of the box on GitHub Actions — push to `main` or open a PR to activate them.

---

## Defensive layers at a glance

### Scanning and analysis

| Workflow | What it does | Findings go to |
|----------|-------------|----------------|
| `codeql.yml` | Static analysis — injection, XSS, insecure deserialization, and more | Security → Code scanning |
| `gitleaks.yml` | Detects secrets and credentials in git history (full depth) | Actions log |
| `trivy.yml` | Scans files, configs, and containers for CVEs, secrets, and misconfigurations | Security → Code scanning |
| `osv-scanner.yml` | Detects known-vulnerable open-source dependencies via the OSV database | Actions log |
| `scorecard.yml` | Measures overall supply-chain security posture (OpenSSF) | Security → Code scanning |
| `checkov.yml` | Policy-as-code scan for GitHub Actions, Dockerfiles, Terraform, Kubernetes | Security → Code scanning |
| `actionlint.yml` | Lints every workflow file for syntax and logic errors | Actions log |
| `zizmor.yml` | Security audit of workflow files: injection, permissions, unsafe patterns | Security → Code scanning |

### Supply-chain and hygiene

| File | What it does |
|------|-------------|
| `sbom.yml` | Generates a SPDX-JSON Software Bill of Materials on every push to `main` |
| `security-hygiene.yml` | Dependency review on PRs + enforces SHA-pinned actions and no `write-all` permissions |
| `dependabot.yml` | Automated dependency update PRs for Actions, npm, pip, and Docker |

### Repository guardrails

| File | What it does |
|------|-------------|
| `.github/CODEOWNERS` | Requires owner review for any change to workflows, docs, or security config |
| `.github/ISSUE_TEMPLATE/security-finding.yml` | Structured triage form for scanner findings |
| `.github/ISSUE_TEMPLATE/vulnerability-triage.yml` | Structured report form for vulnerability disclosures |

### Runner hardening

Every workflow uses `step-security/harden-runner` in `audit` mode, which records all outbound
network calls made by the runner. Tighten to `block` mode with an `allowed-endpoints` list
once you have profiled the expected egress for each workflow.

---

## How to use

1. Push changes to `main` or open a pull request.
2. Go to **GitHub → Actions** — workflows run automatically.
3. Review findings:
   - **Security → Code scanning alerts** — CodeQL, Trivy, Scorecard, Checkov, Zizmor
   - **Actions run logs** — Gitleaks, OSV-Scanner, Actionlint, dependency review
4. Triage critical and high findings first. See `docs/incident-response.md` for step-by-step guidance.
5. Track findings with the structured issue templates.

---

## Documentation

| Doc | Description |
|-----|-------------|
| `docs/incident-response.md` | What to do when each scan fires — triage, contain, remediate, verify |
| `docs/adoption-guide.md` | Per-stack setup guide: Node.js, Python, Docker, Terraform/Kubernetes |
| `docs/hardening-checklist.md` | Full hardening checklist across secrets, deps, CI/CD, IaC, and more |
| `SECURITY.md` | Security policy and how to report vulnerabilities |

---

## Customization

- All workflow triggers include inline comments explaining what to change.
- Dependabot ecosystems can be trimmed to match your actual stack.
- All third-party actions are pinned to full commit SHAs — update pins via `dependabot.yml` (Actions ecosystem).
- Checkov's `framework` list can be narrowed to only the IaC types you use.
- CodeQL's `languages` list should be updated to match your codebase.

---

## Disclaimer

This repository is for **authorized defensive security testing and hardening only**.
Do not use it for unauthorized access, exploitation, or any illegal activity.
