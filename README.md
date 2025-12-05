# Reusable workflows & actions for NDLA

Reusable GitHub Actions workflows and composite actions shared across NDLA repositories. Everything lives under `.github/` and is meant to be referenced via `ndlano/reusable-workflows@<ref>`.

## Contents
- `.github/workflows/release.yaml` – multi-arch release workflow (build + manifest publish).
- `.github/actions/setup-deploy-environment` – prepares runners for NDLA deploy tooling.
- `.github/actions/setup-ndla-login` – logs into AWS ECR via an assumed role and Docker Hub.
- `.github/actions/vercel-deploy` – builds and deploys to Vercel with status updates.

## Reusable workflow: `release.yaml`
Reusable via `workflow_call` with a single required input:

```yaml
jobs:
  release:
    uses: ndlano/reusable-workflows/.github/workflows/release.yaml@main
    with:
      component: <name of component to release>
    secrets: inherit
```

### What it does
- Builds and releases a component for `linux/amd64` and `linux/arm64`, in parallel runners.
- Uses internal `ndla release` command from the `NDLANO/deploy` repo.
- Publishes a multi-arch manifest once both images exist.

## Composite action: `setup-deploy-environment`
Sets up everything needed to run NDLA deploy tooling.

Behavior:
- Checks out `NDLANO/deploy` using the provided GitHub token.
- Installs dependencies of the internal `ndla` CLI tool.
- Delegates auth to `setup-ndla-login` to log into AWS ECR and Docker Hub.

## Composite action: `setup-ndla-login`
Logs into AWS ECR (assumed role) and Docker Hub.

Behavior:
- Assumes `RELEASE_ROLE` via STS using the provided client credentials.
- Uses the temporary credentials to log into ECR in `eu-central-1`.
- Logs into Docker Hub with username/password.

## Composite action: `vercel-deploy`
Builds and deploys the current repo to Vercel and updates commit statuses.

Behavior:
- Installs the Vercel CLI, pulls project config, and marks a pending status `vercel-deploy`.
- Builds (`vercel build`) then deploys with `vercel deploy --prebuilt --archive=tgz`.
- Adds a success status with the deployment URL.
- Aliases: on `master` uses `MASTER_ALIAS`; otherwise `${REPO_NAME}-${{ github.event.number }}.vercel.app`.
- Updates status to point to the alias; posts failure status if any step fails.
