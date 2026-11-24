# TaskFlow

A small sample web app that exists to be released by **[ReleaseGuard](https://github.com/hamdounisabri1/releaseguard)**, a deployment approval and rollback manager. TaskFlow is the demo target: its CI registers releases with ReleaseGuard, and ReleaseGuard decides when each release may be deployed to dev, staging and production.

> 🚧 **Status: placeholder.** App code and workflows arrive in ReleaseGuard Sprint 2 (Jira `SCRUM-35`, `SCRUM-38`, `SCRUM-40`).

## Role in the demo

```text
push to main → ci.yml builds and pushes an image → registers the release with ReleaseGuard (OIDC)
ReleaseGuard approves → dispatches deploy.yml → health checks /health → (unhealthy) → rollback = deploy.yml with an earlier digest
```

## Workflows (planned)

| Workflow | Trigger | What it does |
|---|---|---|
| `.github/workflows/ci.yml` | push / PR | Lint, test, Trivy scan, build, push to GHCR, capture the **image digest**, then register the release at `POST /api/v1/releases/ci` using a **GitHub OIDC token** (no shared secret) |
| `.github/workflows/deploy.yml` | `workflow_dispatch`, only from ReleaseGuard | Inputs `release_id`, `image_digest`, `environment`, `deployment_id`; `run-name: deploy-<deployment_id>`; job `environment: <environment>`. Pulls `ghcr.io/<owner>/taskflow@<digest>`, checks the digest, deploys. Used for **every** environment **and** for rollback |

There is deliberately no separate rollback workflow: a rollback is a new deployment of an earlier, known-good digest through the same `deploy.yml`.

## Rules

- Deploy **by digest** (`sha256:…`), never by a mutable tag.
- Production is protected twice: ReleaseGuard's approval gate, plus a GitHub `production` environment protection rule.
- `/health` can be forced to return HTTP 503 with an environment flag, to demo an unhealthy release and its rollback.
- No secrets in the repo: `.env`, `*.pem` and `*.key` are git-ignored.

Work is tracked in ReleaseGuard's Jira project `SCRUM`.
