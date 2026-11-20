# Pixora – Dockerized Setup

This document describes how to run Pixora (Angular 21 + Django 6 + PostgreSQL)
 with Docker Compose. For the classic local development setup
  (venv + dev servers), see the main [README.md](./README.md).

## Table of Contents

- [Pixora – Dockerized Setup](#pixora--dockerized-setup)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Quickstart](#quickstart)
  - [Build](#build)
  - [Managing the Stack](#managing-the-stack)
  - [Configuration](#configuration)
  - [Architecture](#architecture)
  - [Verification](#verification)
  - [Troubleshooting](#troubleshooting)
  - [Attribution](#attribution)

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/) with the `docker compose`
 plugin
- A working shell (Linux, macOS, or WSL)
- No Python, Node.js, or pnpm installation required – everything runs in
 containers

## Quickstart

```bash
git clone git@github.com:ralfs-devs/pixora.git
cd pixora
cp example.env .env
# edit .env and set the five required variables (see Configuration)
docker compose up --build -d
```

The frontend is then reachable at `http://localhost:4200`, the Django admin at
`http://localhost:4200/admin/`.

## Build

Building the images without starting the containers:

```bash
docker compose build          # both images
docker compose build backend  # only the backend image
```

Both Dockerfiles are multi-stage builds:

- **frontend**: Node 24 builds the Angular app with pnpm; the production stage
 runs nginx and serves the static bundle.
- **backend**: pip installs the production requirements; the production stage
 runs the app as a non-root user via gunicorn. `migrate`, `collectstatic`,
  and the superuser bootstrap run automatically at container start.

## Managing the Stack

```bash
docker compose up -d          # start (detached)
docker compose stop           # stop, containers remain
docker compose restart        # stop + start running containers
docker compose down           # remove containers and networks
docker compose down -v        # also remove volumes – deletes all data
```

> [!WARNING]
> `down -v` wipes the database, uploaded media, and collected static files.
> Data that must survive container recreation lives in named volumes (see
> [Architecture](#architecture)).

## Configuration

All configuration is supplied via `.env` (copy of `example.env`).
(see also the [Quickstart](#quickstart) section)
Required entries must be filled in; optional entries are commented out
and use their defaults unless overridden.

> [!IMPORTANT]
> The five required variables are enforced with `:?` checks – `docker compose`
> aborts before building anything if they are missing.

| Variable                | Required | Description                             |
|-------------------------|----------|-----------------------------------------|
| `SECRET_KEY`            | yes      | Django secret key                       |
| `DB_PASSWORD`           | yes      | PostgreSQL password                     |
| `DJANGO_ADMIN_USER`     | yes      | Bootstrap admin username                |
| `DJANGO_ADMIN_EMAIL`    | yes      | Bootstrap admin email                   |
| `DJANGO_ADMIN_PASSWORD` | yes      | Bootstrap admin password (min. 8 chars) |
| `FRONTEND_PORT`         | no       | Frontend host port, default `4200`      |
| `BACKEND_PORT`          | no       | Backend host port, default `8000`       |
| `DB_NAME`               | no       | PostgreSQL database, default `pixora`   |
| `DB_USER`               | no       | PostgreSQL user, default `pixora`       |
| `DEBUG`                 | no       | Django debug flag, default `False`      |
| `ALLOWED_HOSTS`         | no       | default `localhost,127.0.0.1`           |
| `CORS_ALLOWED_ORIGINS`  | no       | default `http://localhost:4200`         |
| `MAX_IMAGE_SIZE_MB`     | no       | Upload limit in MB, default `5`         |

> [!NOTE]
> `CORS_ALLOWED_ORIGINS` is also used as `CSRF_TRUSTED_ORIGINS`, so the admin
> login works behind the frontend reverse proxy. Keep `DEBUG` on `False` –
> leaving it on in production is unsafe.

## Architecture

| Service  | Image                     | Exposed port |
|----------|---------------------------|--------------|
| frontend | nginx (custom image)      | `4200:80`    |
| backend  | python slim (custom image)| '8000:8000'  |
| db       | postgres:17.11-alpine3.24 | internal     |

Services communicate over the internal Compose network using service names, not
`localhost`: nginx proxies `/api/` and `/admin/` to `http://backend:8000`,
and the backend reaches PostgreSQL at `DB_HOST=db`.
Frontend and backend publish ports to the host (bound to `127.0.0.1`,
see [Configuration](#configuration)); the database has no host binding.
Uploaded media is served by nginx directly from the `media_volume`.

## Verification

1. `docker compose ps` – all three services show `Up ... (healthy)`.
2. Open `http://localhost:4200` – the Pixora UI loads.
3. Register a user, log in, create a post with an image – the image must render
   (proof that frontend → backend → media volume → nginx chain works).
4. Open `http://localhost:4200/admin/` and log in with the `DJANGO_ADMIN_*`
   values.

> [!TIP]
> To verify data persistence, restart or recreate the containers and confirm
> that accounts, posts, and images survive:
>
> ```bash
> docker compose down && docker compose up -d
> ```

## Troubleshooting

- **Port 4200 already in use** – another container holds the port
  (`docker ps --filter publish=4200`).
  Stop it or change the mapping in `compose.yaml`.
- **Backend crash loop with `Permission denied: '/app/staticfiles'`** –
  the named volumes were created before the non-root user existed in the image.
  Reset once with
  `docker compose down -v` and start again.
- **Admin login fails with CSRF error** – the browser origin must be listed in
  `CORS_ALLOWED_ORIGINS` (also used as `CSRF_TRUSTED_ORIGINS`).
  The default covers `http://localhost:4200`.
- **Images/avatars not loading (404)** – nginx serves `/media/` from the
  `media_volume`; verify the frontend service mounts it at
  `/usr/share/nginx/media` (read-only).
- **API returns image URLs pointing to the wrong host/port** – nginx must
  forward the original host header (`proxy_set_header Host $http_host;`),
  otherwise Django generates URLs on the bare hostname without port.

## Attribution

> [!NOTE]
> The frontend and backend application code of Pixora was provided by my
> training company ("Developer Akademie GmbH"); all rights to the application
> code remain with Developer Akademie GmbH.
> The containerization artifacts (Dockerfiles, `compose.yaml`,
> nginx configuration, environment examples, and documentation)
> were created as part of the DevSecOps training program.
