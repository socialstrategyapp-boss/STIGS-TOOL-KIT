# Incident Response Guide

This guide tells you exactly what to do when any scan in this toolkit fires a finding.
Triage fast, fix the right things first, and close the loop.

---

## Triage priority order

| Severity | Target response time |
|----------|---------------------|
| Critical | Immediate — same day |
| High     | 24–48 hours |
| Medium   | Next sprint / 7 days |
| Low / Informational | Best-effort backlog |

---

## 1 — CodeQL fires (static analysis finding)

CodeQL findings appear under **Security → Code scanning alerts**.

**Steps:**
1. Open the alert. Read the rule description and the highlighted code path.
2. Determine if it is a true positive or false positive.
   - False positive? Dismiss with a comment explaining why (`won't fix` or `test code`).
   - True positive? Proceed.
3. Identify the root cause: injection, unsafe deserialization, weak crypto, etc.
4. Patch the code. Re-run CodeQL locally if possible (`codeql database analyze`).
5. Open a PR, reference the alert number, and merge after review.
6. Confirm the alert auto-closes in Code Scanning after the fix is merged.

**Escalation**: If the finding indicates an active exploitable path in a production system, treat it as a Critical and notify the security lead immediately.

---

## 2 — Gitleaks fires (secret detected in git history)

Gitleaks findings appear in the **Actions run log** (and optionally Code Scanning).

**Steps:**
1. Identify the secret type (API key, token, password, certificate, etc.).
2. **Revoke the secret immediately** — do not wait. Assume it is already compromised.
   - Rotate the credential in the service that issued it (AWS, GitHub, Stripe, etc.).
   - Invalidate all active sessions that used it.
3. Remove the secret from git history using `git filter-repo` or BFG Repo Cleaner.
   - Never use `git rebase` or `git reset` alone — these do not rewrite all refs.
   - After history rewrite, force-push to all remote branches with team coordination.
4. Audit access logs of the affected service for unauthorized use during the exposure window.
5. File a security finding issue (or private advisory if the secret was highly sensitive).
6. Add the pattern to your `.gitleaks.toml` allow-list only if it is a confirmed false positive.

**Reference**: [GitHub — removing sensitive data](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

---

## 3 — Trivy fires (vulnerability or misconfiguration)

Trivy findings appear under **Security → Code scanning alerts** (SARIF upload).

### 3a — Vulnerability finding (`vuln` scanner)
1. Check the CVE ID and CVSS score. Look it up at [nvd.nist.gov](https://nvd.nist.gov).
2. Check if a fix is available (`ignore-unfixed: true` already filters out unfixed CVEs).
3. Update the affected dependency to the patched version.
4. If no patch exists: assess exploitability in your specific context. Apply a compensating control (WAF rule, network isolation, input validation) and document the accepted risk.

### 3b — Secret finding (`secret` scanner)
Follow the **Gitleaks** process above.

### 3c — Misconfiguration finding (`misconfig` scanner)
1. Review the misconfiguration rule (e.g., Dockerfile running as root, insecure TLS, missing security headers).
2. Apply the recommended fix from the Trivy rule description.
3. Re-run `trivy fs --scanners misconfig .` locally to verify.

---

## 4 — OSV-Scanner fires (known-vulnerable dependency)

Findings appear in the **Actions run log**.

**Steps:**
1. Identify the package name, version, and CVE / GHSA identifier.
2. Check the OSV entry at [osv.dev](https://osv.dev) for severity and patch availability.
3. Update the dependency: `npm update <pkg>`, `pip install --upgrade <pkg>`, etc.
4. If the dependency cannot be updated (breaking change), assess whether the vulnerable code path is reachable in your application.
5. Lock the patched version in your lockfile and commit.
6. Re-run OSV-Scanner to confirm the finding is cleared.

---

## 5 — OpenSSF Scorecard flags a risk

Scorecard findings appear under **Security → Code scanning alerts**.

Common findings and fixes:

| Scorecard check | Typical fix |
|----------------|-------------|
| `Branch-Protection` | Enable branch protection on `main` — require PR reviews and status checks |
| `Pinned-Dependencies` | Pin all GitHub Actions to full commit SHAs (see existing workflows) |
| `Token-Permissions` | Ensure every workflow uses least-privilege `permissions` block |
| `Dangerous-Workflow` | Remove or restrict `pull_request_target` with write access; never use `${{ github.event.pull_request.head.sha }}` in contexts that allow code execution |
| `SAST` | Ensure CodeQL or equivalent is enabled |
| `Vulnerabilities` | Remediate findings from OSV-Scanner / Dependabot |
| `Signed-Releases` | Sign release tags and artifacts |

---

## 6 — Checkov fires (IaC misconfiguration)

Findings appear under **Security → Code scanning alerts** (SARIF upload).

**Steps:**
1. Open the alert. The rule ID (e.g., `CKV_GHA_1`) links to the Checkov documentation.
2. Read the failing check description and the recommended fix.
3. Apply the fix to the relevant file (workflow, Dockerfile, Terraform, Kubernetes manifest).
4. Re-run `checkov -d .` locally to verify.
5. If a finding is a confirmed false positive, suppress it with a `checkov:skip` comment:
   ```yaml
   # checkov:skip=CKV_GHA_1:reason for skipping
   ```

---

## 7 — Actionlint or Zizmor fires (workflow security issue)

Findings appear in the **Actions run log** (actionlint) or **Code Scanning** (zizmor).

Common issues:

| Issue | Fix |
|-------|-----|
| Unpinned action | Pin to full commit SHA (see existing workflows for examples) |
| Script injection via `${{ github.event.* }}` | Use an intermediate env variable; never interpolate untrusted input directly into `run:` scripts |
| Excessive permissions | Restrict the `permissions` block to only what is needed |
| `pull_request_target` misuse | Avoid checking out PR head code and then running it with write tokens |
| Artifact path traversal | Validate artifact names and paths before use |

---

## 8 — Dependabot opens a PR

Dependabot PRs are pre-triaged. Review and merge according to:
- **Patch / minor**: Auto-approve or fast-track after CI passes.
- **Major version**: Review changelog for breaking changes and test before merging.
- **Security advisory attached**: Treat as **High** and merge within 24–48 hours.

---

## General escalation path

1. **Discovery** — Finding detected by scan or manual review.
2. **Triage** — Confirm true/false positive. Assign severity.
3. **Contain** — If exploitable: revoke credentials, isolate affected systems.
4. **Remediate** — Apply fix, test, merge.
5. **Verify** — Re-run the scan that fired to confirm the finding is resolved.
6. **Document** — Close the issue or advisory with a summary of what happened and how it was fixed.
7. **Retrospect** — Add the failure mode to the hardening checklist if it represents a gap.
