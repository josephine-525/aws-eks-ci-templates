# ci-templates

Shared, reusable GitLab CI job templates for the `demo-org` group. Same philosophy as [`terraform-modules`](../terraform-modules) and [`helm-charts`](../helm-charts): one repo, versioned via git tags, included by whichever app repos need it — the platform owns the shared logic, each app repo owns its own parameters and content.

## What's here

```
ci-templates/
└── build-push.yml   # Reusable Docker build+push template (.build_push_image)
```

## Usage

```yaml
include:
  - project: 'demo-org/ci-templates'
    ref: v1.0.0
    file: '/build-push.yml'

build-backend:
  extends: .build_push_image
  stage: build
  variables:
    DOCKERFILE_PATH: backend/Dockerfile
    BUILD_CONTEXT: backend
    ECR_REPO: demoapp-backend
```

See `build-push.yml`'s own header comment for the full variable list (`DOCKERFILE_PATH`, `BUILD_CONTEXT`, `ECR_REPO`, optional `EXTRA_BUILD_ARGS`) and required CI/CD variables in the including project (`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`/`AWS_DEFAULT_REGION`).

## What's deliberately NOT here

- **Deploy logic** (bumping an app's own `values.yaml` image tag, pushing to git, waiting on ArgoCD sync) — that's app-specific, stays in each app's own `.gitlab-ci.yml`. This repo only owns "how to build and push an image," not "what happens after."
- **Platform bootstrap** (LBC/metrics-server/ArgoCD installs, ArgoCD repo-credential Secrets, applying `gitops`'s own Application/ApplicationSet files) — that's cluster-lifecycle work, not per-app work. It lives in the `gitops` repo's own pipeline, run on its own, not included by any app.

## Versioning

Semver tags (`vMAJOR.MINOR.PATCH`), bumped and pushed manually — same convention as `terraform-modules`/`helm-charts`. Major = breaking change to a template's interface (a required variable renamed/removed), minor = new capability (a new optional variable), patch = fix. Consumers pin an explicit `ref:`, so nothing breaks silently when this repo changes.
