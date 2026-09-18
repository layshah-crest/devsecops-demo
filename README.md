# DevSecOps CI/CD Demo

Intentionally vulnerable Flask app used to demo a full DevSecOps pipeline:
secrets scanning, SAST, dependency/image CVE scanning, Dockerfile linting,
Slack alerts, GitOps deploy with ArgoCD, and admission-time policy blocking
with Kyverno.

**Do not deploy this app anywhere public. It is built to be caught by scanners.**

## What's intentionally broken (and why)

| File | Problem | Caught by |
|---|---|---|
| `app.py` | SQL injection in `/user` | Semgrep |
| `requirements.txt` | Flask 1.1.2, Werkzeug 1.0.1, Jinja2 2.11.3 (real CVEs) | Trivy (fs scan) |
| `Dockerfile` | No `USER` — runs as root | Hadolint + image scan |
| `k8s/deployment.yaml` | No `runAsNonRoot` | Kyverno (blocks at apply time) |
| (added live) `config.py` | Hardcoded AWS key pattern | Gitleaks |

Fixed versions are provided alongside each broken file so you can swap them in
live: `requirements.fixed.txt`, `Dockerfile.fixed`, `k8s/deployment.fixed.yaml`.

## One-time setup (do this before the talk, not live)

See `SETUP.md` for exact commands: repo + GitHub Actions secrets, local
cluster, ArgoCD, Kyverno.

## Suggested run order on stage

1. Explain the pipeline file (`.github/workflows/devsecops.yml`) stage by stage.
2. Push as-is → Trivy + Hadolint fail → Slack alert fires.
3. Add the AWS-key snippet from `LIVE_DEMO_LEAK_SNIPPET.txt` → Gitleaks fails
   → Slack alert fires again.
4. Fix one thing at a time (swap in `requirements.fixed.txt`, `Dockerfile.fixed`,
   remove the leaked key) → push → pipeline goes green.
5. Show ArgoCD UI already synced from `k8s/` → change replica count in Git →
   watch it auto-sync.
6. `kubectl apply -f k8s/deployment.yaml` → Kyverno rejects it (no
   `runAsNonRoot`) → then `kubectl apply -f k8s/deployment.fixed.yaml` →
   succeeds.
