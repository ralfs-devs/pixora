---
toc_max_heading_level: 4
sidebar_position: 2
tags:
  - project
  - automation
  - pixora
  - deployment
  - checklist
sidebar_label: Description
description: An editable, copyable checklist with all tasks for the Pixora
 deployment project.
---

# Pixora Deployment - Checklist

Please work through every item before submitting this project. If you added
 anything beyond these requirements, mention it briefly so your mentor can
  review it when needed.

The Pixora containerization project must be completed and approved before you
 begin this checklist. Use the approved `Containerfile` or `Dockerfile` and
  Compose file from that project.

## 1. Repository

### Workflow files

- [*] A workflow runs tests for pull requests targeting the repository’s default
 branch.
- [*] The test workflow runs again when a new commit is pushed to the pull
 request.
- [*] A build workflow builds an image for pull-request updates targeting the
 default branch.
- [*] A build workflow builds an image when a branch is merged into the default
 branch.
- [*] A build workflow builds an image when a Git tag is created and pushed.
- [*] Tests run successfully before an image is published.
- [*] Images are published to the GitHub Container Registry (GHCR).
- [*] Every Git tag creates an image tag with the same release version.
- [*] No workflow creates or publishes a `latest` tag.
- [*] Image naming is consistent and documented.

### Deployment workflow

- [*] The deployment workflow is available only for Git-tag releases.
- [*] Deployment must be started manually by a developer.
- [*] The workflow transfers the approved Compose file to your own Cloud Server
 or Cloud VM.
- [*] The workflow creates the required remote environment file if it does not
 exist.
- [*] An existing remote environment file is preserved and is not overwritten
 unintentionally.
- [*] The remote machine authenticates to GHCR with a suitable read-only package
 credential where possible.
- [*] The workflow pulls the image belonging to the selected Git tag.
- [*] The workflow starts or updates the Compose setup on your own Cloud Server
 or Cloud VM.
- [*] The workflow fails clearly when transfer, authentication, image pull, or
 startup fails.
- [*] A post-deployment health check or smoke test is performed.

### Cloud Server or Cloud VM

- [*] Your own Cloud Server or Cloud VM has Docker and Docker Compose installed.
- [*] A dedicated deployment user is available.
- [*] Required firewall rules and application ports are configured.
- [*] GitHub Actions can reach the machine using the configured SSH address and
 credentials.
- [*] The deployed service is reachable through the configured address, port,
 or reverse proxy.
- [*] You know how to roll back by deploying an earlier Git-tagged image.

## 2. Documentation

- [*] The README explains the test, build, publish, and deployment workflows.
- [*] The required GitHub secrets and protected environment settings are
 documented by name and purpose.
- [*] The image name and tag format are documented.
- [*] The manual deployment procedure is documented step by step.
- [*] The remote environment-file behavior is documented.
- [*] The post-deployment health check is documented.
- [*] The rollback procedure is documented.
- [*] The documentation contains links to useful workflow runs or other
 evidence.

## 3. Hints

### General hints

- [*] Use the repository’s default branch dynamically where practical, or
 document the assumption clearly.
- [*] Treat the Git tag as the release identity and pass that exact version
 through publishing and deployment.
- [*] Test pull-request, merge, tag, and manual deployment behavior separately.
- [*] Keep deployment configuration separate from application secrets.

### Security hints

- [*] Use explicit least-privilege workflow and job permissions.
- [*] Use protected GitHub environments for deployment approvals and secrets.
- [*] Do not print SSH keys, tokens, passwords, or environment-file contents
 in logs.
- [*] Do not commit private keys, server credentials, or real environment files.
- [*] Review third-party actions and pin them to reviewed versions or commit
 SHAs where appropriate.
- [*] Handle Git tags and shell variables safely.
- [*] Do not deploy to another student’s Cloud Server or Cloud VM.

### Scope hint

- [*] Do not author the `Containerfile` or `Dockerfile` as part of this project.
- [*] Do not author the Compose file as part of this project.
- [*] Do not publish or deploy a `latest` image tag.

## 4. Testing

Before you submit the project, make sure that:

- [*] A pull request runs the tests successfully.
- [*] Pushing another commit to the pull request starts the checks again.
- [*] Merging into the default branch builds an image successfully.
- [*] Creating and pushing a Git tag publishes the matching versioned image to
 GHCR.
- [*] The GHCR package contains no `latest` tag.
- [*] The deployment can be started manually for the selected Git tag.
- [*] The selected image is deployed successfully to your own Cloud Server or
 Cloud VM.
- [*] The remote environment file is created when missing.
- [*] An existing remote environment file remains unchanged.
- [*] The deployed application passes its health check or smoke test.
- [*] A failed check prevents publishing or deployment as intended.

## What is not a TODO here

- [*] Do not repeat the containerization project.
- [*] Do not deploy to another student’s machine.
- [*] Do not store secrets in the repository, workflow files, or container
 image.

## What you must deliver

- [*] Working GitHub Actions workflows for testing, image building, publishing,
 and deployment.
- [*] Deployment documentation and safe environment-file examples.
- [*] A short explanation of design decisions and rollback.
- [*] Evidence of the pull-request run, merge build, tagged GHCR image, and
 successful manual deployment.

## Helpful links

- [GitHub Actions events](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [GitHub Actions workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Publishing Docker images to GHCR](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)
- [GitHub Actions secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
- [Docker Compose environment variables](https://docs.docker.com/compose/how-tos/environment-variables/)
