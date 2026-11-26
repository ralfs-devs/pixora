# Pixora – CI/CD & Deployment

This document describes the automated pipeline (testing, image publishing,
deployment to the cloud VM) and the manual procedures around it.
For the local development setup see the main [README.md](./README.md),
for the Docker Compose stack see [DOCKER_README.md](./DOCKER_README.md),
for attribution of the Developer Akademie GmbH material see
[DOCKER_README.md – Attribution](./DOCKER_README.md#attribution).

## Table of Contents

- [Pixora – CI/CD & Deployment](#pixora--cicd--deployment)
  - [Table of Contents](#table-of-contents)
  - [Pipeline Overview](#pipeline-overview)
  - [Workflows](#workflows)
  - [Image Naming and Tags](#image-naming-and-tags)
  - [Deployment Target](#deployment-target)
  - [Remote Environment File](#remote-environment-file)
  - [Manual Deployment](#manual-deployment)
  - [Post-Deployment Smoke Test](#post-deployment-smoke-test)
  - [Rollback](#rollback)
  - [GitHub Secrets](#github-secrets)
  - [Evidence](#evidence)

## Pipeline Overview

Every push to `main` triggers exactly one workflow
run that chains all stages:

push ▶ Testing ▶ Build & Publish ▶ Deploy ▶ Smoke Test

- **Testing** runs first; on failure the pipeline stops and nothing is
  built, published, or deployed.
- **Build & Publish** publishes both images to the GitHub Container Registry
  (GHCR); the backend image must pass a socket health check before the
  frontend image is built.
- **Deploy** serializes via a concurrency group, so deployments never
  overlap.

## Workflows

| Workflow file  | Purpose                                                    |
|----------------|------------------------------------------------------------|
| `testing.yaml` | Backend + frontend test jobs inherited from original Repo  |
| `ci_cd.yaml`   | Orchestration: testing stage, build, publish, deploy       |

`ci_cd.yaml` includes `testing.yaml` via `workflow_call`, so one single run
covers the whole chain for a commit.

## Image Naming and Tags

Images are published as `ghcr.io/ralfs-devs/pixora-backend:<ref>` and
`ghcr.io/ralfs-devs/pixora-frontend:<ref>` where `<ref>` is the commit SHA
(and additionally the Git tag name on `rev-*` tag pushes).

Published to GHCR under the repository owner namespace; no `latest` tag is
ever created:

| Ref pushed            | Image tags                                        |
|-----------------------|---------------------------------------------------|
| branch push           | commit SHA                                        |
| tag push (`rev-*`)    | commit SHA **and** tag name (e.g. `rev-abc1234`)  |

On the VM, images are pulled by SHA and retagged to
`pixora-backend:local` / `pixora-frontend:local`, which are the only image
references the Compose file uses. For the base images and the overall stack
composition see
[DOCKER_README.md – Architecture](./DOCKER_README.md#architecture).

## Deployment Target

- **VM**: cloud server, reached from the GitHub runner via SSH/scp using the
  `production` environment (no required reviewers – the pipeline is fully
  automatic by design).
- **Deploy user**: `pixora-deploy`, a dedicated user with Docker group
  membership only (least privilege).
- **Target directory**: `/home/ralf-hamacher/pixora` with
  `docker-compose.yaml` and `.env`.
- **GHCR authentication**: the VM authenticates with a read-only package
  credential (`read:packages` PAT) stored under the deploy user via
  `docker login` – it can pull images but never push.
- **Data safety**: the pipeline updates the stack with
  `docker compose up -d --remove-orphans` and never removes volumes;
  database content and uploads survive every deployment. Migration behavior
  at container startup is described in
  [DOCKER_README.md – Build](./DOCKER_README.md#build).

## Remote Environment File

The workflow creates `.env` in the target directory **only if it does not
exist yet**; an existing file is never overwritten (the run log states
`.env exists - preserving` in that case). Configuration changes therefore
require editing the file on the VM (or deleting it once so the next
deployment recreates it from the secrets).

Which variables the file contains and what they mean is documented in
[DOCKER_README.md – Configuration](./DOCKER_README.md#configuration).

## Manual Deployment

Without a push, the pipeline can be started from the GitHub UI:

1. Open **Actions** → **CI/CD**.
2. Click **Run workflow** and choose the branch (`main` or
   `feature/deployment`).
3. The full chain runs for the latest commit of that branch.

## Post-Deployment Smoke Test

Every deployment ends with two automated checks on the VM:

- `http://localhost:4200` – frontend reachable
- `http://localhost:4200/admin/` – backend reachable via the frontend proxy

If either check fails, the workflow fails visibly. For interactive
verification beyond that see
[DOCKER_README.md – Verification](./DOCKER_README.md#verification).

## Rollback

Rolling back means redeploying an earlier, known-good state of the
application:

1. Open **Actions** → **CI/CD** and locate the last green run of the
   commit you want to return to.
2. Click **Re-run all jobs**. The pipeline re-executes for exactly this
   commit SHA: testing, image build, and deployment to the VM run again
   and the VM ends up on the old state. Database volumes are never touched
   by the pipeline, so data survives the rollback.

Alternatively, the images of a `rev-*` Git tag (see
[Image Naming and Tags](#image-naming-and-tags)) can be deployed the same
way, since every tag is also a green pipeline run.

> [!NOTE]
> A rollback restores application code; reverting applied database
> migrations is a manual Django task and outside the pipeline's scope.

## GitHub Secrets

All secrets live in the protected `production` environment.  
These three are pipeline-specific:

| Secret             | Purpose                                       |
|--------------------|-----------------------------------------------|
| `SSH_HOST`         | VM address used by the runner for SSH and scp |
| `SSH_USER`         | SSH login user (the deploy user)              |
| `SSH_PRIVATE_KEY`  | Private SSH key of the deploy user            |

> [!NOTE]
> The remaining secrets carry the application configuration values that
> seed the remote `.env`. For details see
> [DOCKER_README.md – Configuration](./DOCKER_README.md#configuration).

Publishing to GHCR uses the workflow's built-in `GITHUB_TOKEN` with
least-privilege permissions (`contents: read`, `packages: write`) – no
secret configuration required for that.

## Evidence

Example workflow runs demonstrating the documented behavior:

| Demonstrated behavior       | Run    |
|-----------------------------|--------|
| Push deployment             | [PD]   |
| Tag push / versioned images | [TP]   |
| Manual deployment (re-run)  | [MD]   |

[PD]: https://github.com/ralfs-devs/pixora/actions/runs/45677553603
[TP]: https://github.com/ralfs-devs/pixora/actions/runs/45682570414
[MD]: https://github.com/ralfs-devs/pixora/actions/runs/45677553603/attempts/2
