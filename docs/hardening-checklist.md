# Hardening Checklist

Use this checklist to baseline and continuously improve defensive security posture.

## 1) Secrets hygiene

- [ ] Enable secret scanning in CI (Gitleaks and/or platform-native scanning)
- [ ] Block commits that contain tokens, keys, or credentials
- [ ] Rotate exposed secrets immediately upon detection
- [ ] Store secrets in a dedicated secrets manager, not source control
- [ ] Redact sensitive values from logs and error messages
- [ ] Audit git history for historical secret exposure after any leak

## 2) Dependency hygiene

- [ ] Enable Dependabot updates for all active ecosystems
- [ ] Run OSV-Scanner (or equivalent) on pull requests and schedules
- [ ] Pin dependencies where practical and review major-version upgrades
- [ ] Remove unmaintained or unused dependencies
- [ ] Generate and store an SBOM (Software Bill of Materials) for every release
- [ ] Validate SBOM against known CVE feeds on a schedule

## 3) CI/CD permissions hardening

- [ ] Use least-privilege `permissions` in every workflow
- [ ] Avoid `write-all` permissions unless absolutely required
- [ ] Require pull-request review before workflow changes are merged
- [ ] Pin all third-party GitHub Actions to full commit SHAs (not floating tags)
- [ ] Restrict secret access for workflows from forks
- [ ] Use `step-security/harden-runner` to audit or restrict runner egress
- [ ] Audit workflows with `actionlint` and `zizmor` on every workflow change
- [ ] Scan workflow files with Checkov for policy violations

## 4) Repository guardrails

- [ ] Enable branch protection on `main`: require PR reviews + status checks
- [ ] Require Code Owner approval via `CODEOWNERS` for security-sensitive paths
- [ ] Require signed commits for all merges to `main`
- [ ] Disable force-push to protected branches
- [ ] Set a workflow concurrency limit to prevent parallel run abuse
- [ ] Use `ISSUE_TEMPLATE` forms to structure security finding reports

## 5) Supply-chain security

- [ ] Run OpenSSF Scorecard and address all High/Critical findings
- [ ] Pin base container images to a specific SHA digest
- [ ] Verify release integrity with cryptographic signatures or SLSA provenance
- [ ] Use `actions/dependency-review-action` to block high-severity deps in PRs
- [ ] Monitor for abandoned or typosquatted packages in your dependency graph

## 6) Web app hardening basics

- [ ] Enforce HTTPS and modern TLS configurations
- [ ] Add CSP, HSTS, X-Frame-Options, and X-Content-Type-Options headers
- [ ] Validate and sanitize untrusted input server-side
- [ ] Protect sessions (secure cookies, expiration, invalidation)
- [ ] Test for XSS, CSRF, IDOR, and authentication bypass paths

## 7) Mobile app hardening basics

- [ ] Store sensitive data in secure platform keystores only
- [ ] Disable debug and verbose logging in release builds
- [ ] Protect API traffic with proper TLS validation
- [ ] Prevent plaintext secrets in app bundles and local caches
- [ ] Review backup/export behavior for sensitive data leakage

## 8) Server hardening basics

- [ ] Apply OS and package updates on a fixed cadence
- [ ] Disable unused services and close unused network ports
- [ ] Enforce least privilege for service accounts and admins
- [ ] Enable centralized logging and anomaly alerting
- [ ] Verify backup encryption and restore integrity regularly

## 9) IaC and container hardening

- [ ] Scan Terraform, Kubernetes, and Dockerfile configs with Checkov or equivalent
- [ ] Run containers as non-root with a read-only root filesystem
- [ ] Drop all Linux capabilities and add back only what is required
- [ ] Set CPU and memory resource limits on all containers
- [ ] Apply Kubernetes NetworkPolicies to restrict lateral movement
- [ ] Use Terraform remote state with encryption and access controls

## 10) Logging and telemetry leakage checks (encrypted apps)

- [ ] Confirm plaintext message/content never appears in logs
- [ ] Review crash reports for keys, tokens, or decrypted payloads
- [ ] Minimize metadata exposure (identifiers, timestamps, payload sizes)
- [ ] Ensure analytics SDKs do not collect sensitive user content
- [ ] Validate notification previews do not leak confidential data

## 11) Incident response readiness

- [ ] Document what to do when each scan type fires (see `docs/incident-response.md`)
- [ ] Establish a triage severity matrix and response-time SLA
- [ ] Run a tabletop exercise: simulate a leaked secret and trace the full response
- [ ] Confirm the private security advisory flow is configured and tested
- [ ] Review and update this checklist after every security incident
