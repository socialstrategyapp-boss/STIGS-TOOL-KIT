# Stigs tool box

Stigs tool box is a defensive security and hardening toolbox for repository, CI/CD, and supply-chain hygiene.

It provides a practical baseline you can use immediately in GitHub for authorized security testing, secure development, and hardening workflows.

## Included defensive tooling

- **CodeQL analysis** (`.github/workflows/codeql.yml`)
  - Static code analysis for common security weaknesses.
- **Gitleaks secret scanning** (`.github/workflows/gitleaks.yml`)
  - Detects committed secrets and exposed credentials.
- **Trivy repository scan** (`.github/workflows/trivy.yml`)
  - Scans filesystem content for vulnerabilities, secrets, and misconfigurations.
- **OSV-Scanner dependency scan** (`.github/workflows/osv-scanner.yml`)
  - Detects known vulnerabilities in open-source dependencies.
- **OpenSSF Scorecard** (`.github/workflows/scorecard.yml`)
  - Measures repository supply-chain security posture and best practices.
- **Security hygiene workflow** (`.github/workflows/security-hygiene.yml`)
  - Adds dependency review and a simple workflow-permissions safeguard.
- **Dependabot updates** (`.github/dependabot.yml`)
  - Automated dependency update PRs for common ecosystems.

## How to use

1. Push changes to `main` or open a pull request against `main`.
2. Go to **GitHub → Actions** and run workflows manually or let scheduled triggers run.
3. Review findings in:
   - **Security → Code scanning alerts** (for SARIF uploads: CodeQL/Trivy/Scorecard)
   - **Workflow run logs** (for Gitleaks, OSV-Scanner, dependency review)
4. Triage findings and remediate high/critical issues first.

### Customization notes

- Workflow triggers include comments so you can customize branch scope and schedules.
- Dependabot includes common ecosystems and can be trimmed to match your stack.
- All workflows are configured with least-privilege permissions where practical.

## Disclaimer

This repository is for **authorized defensive security testing and hardening only**.
Do not use it for unauthorized access, exploitation, or any illegal activity.
