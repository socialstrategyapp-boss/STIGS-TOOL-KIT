# Hardening Checklist

Use this checklist to baseline and continuously improve defensive security posture.

## 1) Secrets hygiene

- [ ] Enable secret scanning in CI (Gitleaks and/or platform-native scanning)
- [ ] Block commits that contain tokens, keys, or credentials
- [ ] Rotate exposed secrets immediately
- [ ] Store secrets in a dedicated secrets manager, not source control
- [ ] Redact sensitive values from logs and error messages

## 2) Dependency hygiene

- [ ] Enable Dependabot updates for all active ecosystems
- [ ] Run OSV-Scanner (or equivalent) on pull requests and schedules
- [ ] Pin dependencies where practical and review major-version upgrades
- [ ] Remove unmaintained or unused dependencies
- [ ] Track SBOM outputs for release artifacts

## 3) CI/CD permissions hardening

- [ ] Use least-privilege `permissions` in every workflow
- [ ] Avoid `write-all` permissions unless absolutely required
- [ ] Require pull-request review before workflow changes are merged
- [ ] Prefer official or well-maintained actions and pin versions/tags responsibly
- [ ] Restrict secret access for workflows from forks

## 4) Web app hardening basics

- [ ] Enforce HTTPS and modern TLS configurations
- [ ] Add CSP, HSTS, X-Frame-Options, and X-Content-Type-Options headers
- [ ] Validate and sanitize untrusted input server-side
- [ ] Protect sessions (secure cookies, expiration, invalidation)
- [ ] Test for XSS, CSRF, IDOR, and authentication bypass paths

## 5) Mobile app hardening basics

- [ ] Store sensitive data in secure platform keystores only
- [ ] Disable debug and verbose logging in release builds
- [ ] Protect API traffic with proper TLS validation
- [ ] Prevent plaintext secrets in app bundles and local caches
- [ ] Review backup/export behavior for sensitive data leakage

## 6) Server hardening basics

- [ ] Apply OS and package updates on a fixed cadence
- [ ] Disable unused services and close unused network ports
- [ ] Enforce least privilege for service accounts and admins
- [ ] Enable centralized logging and anomaly alerting
- [ ] Verify backup encryption and restore integrity regularly

## 7) Logging and telemetry leakage checks (encrypted apps)

- [ ] Confirm plaintext message/content never appears in logs
- [ ] Review crash reports for keys, tokens, or decrypted payloads
- [ ] Minimize metadata exposure (identifiers, timestamps, payload sizes)
- [ ] Ensure analytics SDKs do not collect sensitive user content
- [ ] Validate notification previews do not leak confidential data
