# Adoption Guide

Copy the workflows and configuration that match your stack. Every section below
lists which files are **baseline** (recommended for everyone) and which are
**optional** (add when the technology is present in your repo).

---

## Baseline — all repositories

These tools are stack-agnostic. Enable them in every repo.

| File | Purpose |
|------|---------|
| `.github/workflows/codeql.yml` | Static analysis — catches injection, XSS, insecure deserialization, etc. |
| `.github/workflows/gitleaks.yml` | Detects secrets and credentials committed to git history |
| `.github/workflows/trivy.yml` | Scans files, configs, and containers for CVEs and misconfigurations |
| `.github/workflows/osv-scanner.yml` | Detects known-vulnerable open-source dependencies |
| `.github/workflows/scorecard.yml` | Measures overall supply-chain security posture |
| `.github/workflows/security-hygiene.yml` | Dependency review on PRs + workflow permissions check |
| `.github/workflows/actionlint.yml` | Lints GitHub Actions workflows for syntax and logic errors |
| `.github/workflows/zizmor.yml` | Security audit of GitHub Actions workflows (injection, permissions, etc.) |
| `.github/workflows/checkov.yml` | Policy-as-code scan for IaC, workflows, Dockerfiles |
| `.github/workflows/sbom.yml` | Generates a Software Bill of Materials on every push to main |
| `.github/dependabot.yml` | Automated dependency update PRs |
| `.github/CODEOWNERS` | Enforces review from the security owner on sensitive paths |
| `.github/ISSUE_TEMPLATE/` | Structured templates for findings and vulnerability reports |

---

## JavaScript / Node.js projects

### Extra steps
1. Ensure `package-lock.json` or `yarn.lock` is committed.
   OSV-Scanner and Dependabot need the lockfile to detect vulnerabilities accurately.

2. **Enable npm ecosystem in `dependabot.yml`** (already included — remove the block if not needed).

3. **Expand CodeQL language list** in `codeql.yml`:
   ```yaml
   languages: javascript-typescript
   ```
   This is already set. If you also have Python, add `python` separated by a comma.

4. **Add a licence check** (optional):
   ```bash
   npx license-checker --onlyAllow 'MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC'
   ```
   Run this as a CI step to prevent copyleft licenses entering your dependency tree.

5. **Harden your npm scripts**: avoid `--unsafe-perm`, pin `node` engine in `package.json`,
   and do not use `npm install --global` in CI.

### Trivy configuration note
When a `package-lock.json` is present, Trivy will scan it automatically under `vuln` scanner mode.

---

## Python projects

### Extra steps
1. Commit a pinned `requirements.txt` or use `pip-compile` to lock transitive deps.
   Add `requirements.txt` to OSV-Scanner:
   ```yaml
   scan-args: |-
     --lockfile=requirements.txt
   ```

2. **Enable pip ecosystem in `dependabot.yml`** (already included).

3. **Expand CodeQL language list** in `codeql.yml`:
   ```yaml
   languages: python
   ```

4. **Add Bandit** (optional — Python-specific SAST):
   ```yaml
   - name: Run Bandit
     run: |
       pip install bandit
       bandit -r . -f json -o bandit.json || true
   ```
   Bandit catches common Python security issues (SQL injection, shell injection, hardcoded passwords).

5. **Virtual environment hygiene**: never commit `.venv/`, `__pycache__/`, or `.pyc` files.
   Add them to `.gitignore`.

---

## Container / Docker projects

### Extra steps
1. **Add Docker ecosystem to `dependabot.yml`** (already included — targets `Dockerfile` in `/`).
   If your Dockerfiles live in subdirectories, add extra entries:
   ```yaml
   - package-ecosystem: "docker"
     directory: "/services/api"
     schedule:
       interval: "weekly"
   ```

2. **Enable Trivy container image scan** — add a job to `trivy.yml`:
   ```yaml
   trivy-image:
     name: Scan container image
     runs-on: ubuntu-latest
     steps:
       - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
       - name: Build image
         run: docker build -t myapp:${{ github.sha }} .
       - name: Scan image with Trivy
         uses: aquasecurity/trivy-action@ed142fd0673e97e23eac54620cfb913e5ce36c25 # v0.36.0
         with:
           image-ref: myapp:${{ github.sha }}
           format: sarif
           output: trivy-image.sarif
           severity: CRITICAL,HIGH
       - uses: github/codeql-action/upload-sarif@dd903d2e4f5405488e5ef1422510ee31c8b32357 # v3
         with:
           sarif_file: trivy-image.sarif
   ```

3. **Dockerfile best practices**:
   - Use a minimal base image (`distroless`, `alpine`, `slim`).
   - Run as a non-root user (`USER 1001`).
   - Avoid `COPY . .` — be explicit about what you copy.
   - Use multi-stage builds to keep final images lean.
   - Pin base image tags to a specific SHA digest:
     ```dockerfile
     FROM node:20.18.0-alpine3.20@sha256:<digest>
     ```

4. **SBOM for container images** — extend `sbom.yml` to scan the built image:
   ```yaml
   - uses: anchore/sbom-action@e22c389904149dbc22b58101806040fa8d37a610 # v0.24.0
     with:
       image: myapp:${{ github.sha }}
       format: spdx-json
       artifact-name: sbom-image.spdx.json
   ```

---

## Infrastructure-as-Code (Terraform / Kubernetes / CloudFormation)

### Extra steps
1. **Checkov** is already enabled and scans Terraform, Kubernetes, and CloudFormation
   automatically when those files are present in the repo.

2. **Recommended Terraform hardening** (Checkov catches most of these):
   - Enable state file encryption (`backend "s3"` with `encrypt = true`).
   - Use IAM roles with least-privilege policies.
   - Block public S3 buckets (`aws_s3_bucket_public_access_block`).
   - Enable CloudTrail, GuardDuty, and Security Hub where applicable.
   - Pin provider versions in `required_providers`.

3. **Kubernetes hardening** (Checkov catches most of these):
   - Set `securityContext.runAsNonRoot: true`.
   - Set `readOnlyRootFilesystem: true`.
   - Drop all capabilities: `capabilities.drop: ["ALL"]`.
   - Set resource limits for all containers.
   - Use NetworkPolicies to restrict pod-to-pod traffic.
   - Never store secrets in ConfigMaps — use Kubernetes Secrets or an external secrets manager.

4. **SLSA provenance for Terraform modules** (optional advanced):
   Use `slsa-framework/slsa-github-generator` to generate L3 build provenance for any
   released Terraform module or container image. See the
   [SLSA generator docs](https://github.com/slsa-framework/slsa-github-generator) for setup.

5. **Infra drift detection** (optional):
   Schedule a `terraform plan` in CI and alert on unexpected drift between the
   desired state and the live cloud environment.

---

## Enabling branch protection (manual step in GitHub UI)

No workflow can enforce branch protection — it must be configured in the repository settings.

Go to **Settings → Branches → Add rule** for `main` and enable:
- [x] Require a pull request before merging
- [x] Require approvals (minimum 1)
- [x] Dismiss stale pull request approvals when new commits are pushed
- [x] Require review from Code Owners
- [x] Require status checks to pass before merging
  - Add: `Analyze`, `Detect Secrets`, `Scan Filesystem and Config`, `Scan Dependencies`
- [x] Require branches to be up to date before merging
- [x] Do not allow bypassing the above settings

For high-security repos also enable:
- [x] Require signed commits
- [x] Require linear history

---

## Suggested label set

Create these labels in your repository for consistent triage:

| Label | Color | Use |
|-------|-------|-----|
| `security` | `#d73a4a` | Any security-related issue or PR |
| `finding` | `#e4e669` | Automated scanner finding |
| `vulnerability` | `#b60205` | Confirmed exploitable vulnerability |
| `false-positive` | `#0075ca` | Dismissed finding |
| `accepted-risk` | `#cfd3d7` | Known issue accepted with justification |
| `dependencies` | `#0075ca` | Dependabot or dependency-related |
| `critical` | `#b60205` | Severity: Critical |
| `high` | `#e11d48` | Severity: High |
| `medium` | `#f97316` | Severity: Medium |
| `low` | `#eab308` | Severity: Low |
