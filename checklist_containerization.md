---
toc_max_heading_level: 4
sidebar_position: 2
tags:
  - project
  - checklist
sidebar_label: Checklist
description: An editable, copyable checklist with all tasks for the Pixora
 containerization project.
---

# Pixora Containerization - Checklist

Please work through every item before submitting this project. If you added
 anything beyond these requirements, mention it briefly so your mentor can
  review it when needed.

## 1. Repository

### Existing files

- [*] A `.gitignore` file exists and excludes irrelevant files from the Git
 repository.
- [*] You use exactly one image definition: `Containerfile` or `Dockerfile`.
- [*] You use exactly one Compose definition: `compose.yaml` or
 `docker-compose.yaml`.
- [*] A `.dockerignore` file exists and excludes irrelevant files from the
 image build context.
- [*] An environment-file example named `example.env` exists and contains
 placeholders only.

### Containerfile / Dockerfile

- [*] The base images use explicit version tags and do not use `latest`.
- [*] The image installs only the dependencies required to run Pixora.
- [*] The corresponding build and run stages in the containerfile are clearly
 separated. This applies to frontend and backend.
- [*] The working directory, exposed ports, and startup commands are configured
 correctly.
- [*] The image does not contain Git data, local secrets, virtual environments,
 dependency caches, or unnecessary build artifacts.
- [*] The application runs as a non-root user where practical.
- [*] The image can be built locally without errors.

### Compose file

- [*] The Compose file defines the required Pixora services, including the
 `frontend` and `backend`.
- [*] The Compose file defines a dedicated PostgreSQL service (`db`) as the
 database. SQLite is not allowed.
- [*] The PostgreSQL image uses an explicit version tag, for example
 `postgres:17-alpine`.
- [*] The backend connects to PostgreSQL via the Compose service name, not
 `localhost`.
- [*] Database name, user, and password are supplied through environment
 variables and are not hard-coded.
- [*] The PostgreSQL data directory is stored in a named volume.
- [*] There is no port binding (`ports:`) for the `db` service.
- [*] The backend starts only after PostgreSQL is ready, for example via a
 `pg_isready` health check and `depends_on` with `condition: service_healthy`.
- [*] Service names and internal service-to-service communication are configured
 correctly.
- [*] Required ports are mapped so the application can be reached from the host.
 The host-side ports are configurable via environment variables and default to
  the Pixora ports (frontend `4200`, backend `8000`).
- [*] Required networks, volumes, and service dependencies are configured.
- [*] Persistent data is stored in a volume where it must survive container
 recreation.
- [*] Configuration is supplied through environment variables or an environment
 file.
- [*] No passwords, tokens, private keys, or other secrets are hard-coded.
- [*] Health checks or suitable startup dependencies are configured where
 needed.

## 2. Documentation

- [*] The README explains how to build the image locally.
- [*] The README explains how to start, stop, restart, and remove the Compose
 setup.
- [*] The README explains which environment variables are required, including
 the PostgreSQL variables.
- [*] The README explains how to verify that the frontend and backend are
 working.
- [*] The README documents exposed ports, volumes, and important service names.
- [*] The documentation is written clearly enough for another student to run
 Pixora without additional setup.
- [*] The documentation does not include real credentials or other sensitive
 information.

## 3. Hints

### General hints

- [*] Use one consistent naming scheme. Do not create both `Dockerfile` and
 `Containerfile`, or both Compose filenames.
- [*] Prefer explicit image and dependency versions so the setup is
 reproducible.
- [*] Check the logs of each service when the setup does not start correctly.
- [*] Remember that a service name is normally used for communication between
 Compose services, not `localhost`.
- [*] Remove any SQLite database file (for example `db.sqlite3`) from the
 repository and exclude it via `.gitignore` and `.dockerignore`.

### Security hints

- [*] Do not copy `.env` files containing real secrets into an image.
- [*] Do not commit passwords, tokens, private keys, or cloud-server
 credentials.
- [*] Expose only the ports that are required for local use.

### Scope hint

- [*] Do not create GitHub Actions workflows, GHCR publishing, automated image
 builds, or Cloud Server deployment as part of this project.

## 4. Testing

Before you submit the project, make sure that:

- [*] The image builds successfully from the authored `Containerfile` or
 `Dockerfile`.
- [*] The complete setup starts successfully with Docker Compose.
- [*] All expected containers become ready or healthy.
- [*] The frontend is reachable through the configured host port.
- [*] The frontend can communicate with the backend.
- [*] Database migrations run successfully against PostgreSQL.
- [*] A basic application smoke test succeeds.
- [*] Restarting or recreating containers preserves data that should be
 persistent. Data stored in PostgreSQL survives `docker compose down` and
  `docker compose up`.
- [*] The setup can be stopped and started again successfully.

## What you must deliver

- [*] A working `Containerfile` or `Dockerfile`.
- [*] A working `compose.yaml` or `docker-compose.yaml`.
- [*] A suitable `.dockerignore` file.
- [*] A safe environment-file example.
- [*] Documentation for build, startup, shutdown, verification, and
 troubleshooting.
- [*] Evidence that the setup was tested successfully.

## Helpful links

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Docker build best practices](https://docs.docker.com/build/building/best-practices/)
- [Docker Compose environment variables](https://docs.docker.com/compose/how-tos/environment-variables/)
