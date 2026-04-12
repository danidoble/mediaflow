# MediaFlow

Full-stack media conversion platform. Converts images to WebP/AVIF and transcodes video, with async processing via Celery workers.

All services run behind a single nginx reverse proxy — no extra ports to expose. Everything goes through **port 80** (or whatever you map `PUBLIC_APP_PORT` to).

## Services

| Service | Path | Description |
|---------|------|-------------|
| Web | `http://localhost/` | Frontend (Astro + nginx) |
| API | `http://localhost/api/v1/` | REST API (FastAPI) |
| API Docs | `http://localhost/api/v1/docs` | OpenAPI / Swagger UI |
| MinIO Console | `http://localhost:9003` | Object storage admin (internal only) |

## Requirements

- Docker with the Compose plugin

---

## Quick start — Docker Hub (no build required)

The pre-built images `danidoble/mediaflow-api` and `danidoble/mediaflow-web` are published on Docker Hub. You only need the three files in this folder — no source code required.

### Option A — clone just this folder (sparse checkout)

```shell
git clone --filter=blob:none --sparse https://github.com/danidoble/mediaflow.git
cd mediaflow
git sparse-checkout set mediaflow
cd mediaflow
```

### Option B — download the files directly

```shell
mkdir mediaflow && cd mediaflow

curl -fsSL https://raw.githubusercontent.com/danidoble/mediaflow/main/mediaflow/docker-compose.yml -o docker-compose.yml
curl -fsSL https://raw.githubusercontent.com/danidoble/mediaflow/main/mediaflow/nginx.conf       -o nginx.conf
curl -fsSL https://raw.githubusercontent.com/danidoble/mediaflow/main/mediaflow/.env.example     -o .env.example
```

### 1. Configure environment

```shell
cp .env.example .env
```

Edit `.env` if you need to change ports, credentials, or CORS origins.

Key variables:

| Variable | Default | Notes |
|---|---|---|
| `PUBLIC_APP_PORT` | `80` | Host port for the web + API proxy |
| `MINIO_PUBLIC_ENDPOINT` | `localhost` | `hostname[:port]` exactly as the browser reaches the app, no scheme. Port 80 can be omitted; **any other port must be included** (e.g. `localhost:8080`, `example.com:8443`) |
| `SECRET_KEY` | — | **Change in production** — `openssl rand -hex 32` |
| `POSTGRES_PASSWORD` | `mediaflow` | **Change in production** |
| `MINIO_SECRET_KEY` | `minioadmin` | **Change in production** |

### 2. Pull the images

```shell
docker compose pull
```

### 3. Start the stack

```shell
docker compose up -d
```

All services start in the correct order. Database migrations run automatically via the `migrate` service before the API starts — no manual steps required.

Open `http://localhost` (or the host and port you configured).

---

## Updating to the latest version

```shell
docker compose pull
docker compose up -d
```

---

## Scaling workers

To run more Celery workers for heavier workloads:

```shell
docker compose up -d --scale worker=4
```

## Stopping the stack

```shell
docker compose down          # stop containers, keep volumes
docker compose down -v       # stop containers and delete all data
```

