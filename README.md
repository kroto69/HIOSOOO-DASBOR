# HIOSOOO Dashboard

Dashboard and REST API for managing OLT devices. The repository contains:

- a **Go backend** (`olt-api`)
- a **React + Vite frontend** (`frontend/`)
- a **SQLite** database stored as a local file

## Tech stack

| Layer | Stack |
| --- | --- |
| Backend | Go 1.21, Gin, GORM, Viper |
| Frontend | React, Vite |
| Database | SQLite |
| Local container setup | Docker Compose |

## Project structure

```text
.
├── cmd/server/            # backend entrypoint
├── internal/              # handlers, services, middleware, config
├── frontend/              # React + Vite app
├── configs/config.yaml    # optional YAML config
├── docker-compose.yml     # local development stack
├── docker-compose.prod.yml # production-oriented stack
├── Dockerfile.backend     # backend production image
├── Dockerfile.backend.dev # backend dev image
└── run.sh                 # local native dev helper
```

## Quick start

### Option A — Docker Compose (recommended for local development)

This repository ships with a **development-oriented** Docker Compose setup.
It runs:

- the backend with `go run ./cmd/server/main.go`
- the frontend with the Vite dev server

1. Copy the example environment file:

   ```bash
   cp .env.example .env
   ```

2. Start the stack:

   ```bash
   docker compose up --build
   ```

3. Open the app:

   - Frontend: `http://localhost:5173`
   - Backend API: `http://localhost:3000`
   - Health check: `http://localhost:3000/health`

If port `5173` is already in use, override it before starting Compose:

```bash
FRONTEND_PORT=4173 docker compose up --build
```

If you change the backend port, also update `VITE_API_BASE_URL` in `.env` so the frontend points to the correct API URL.

#### Useful Docker commands

```bash
# start in background
docker compose up -d

# stop containers
docker compose down

# stop and remove volumes
docker compose down -v
```

Use `docker compose down -v` when frontend dependencies become stale and you want Docker to recreate the `node_modules` volume from scratch.

### Option B — Production-oriented Docker Compose

This repository also includes a **production-oriented** container setup.
It differs from the development stack in a few important ways:

- the backend runs as a compiled binary
- the frontend is built into static assets and served by nginx
- the frontend proxies `/api/*` and `/health` to the backend
- only the frontend web port is published publicly by default

1. Create a production environment file:

   ```bash
   cp .env.production.example .env.production
   ```

2. Update the required secrets before first start:

   ```env
   AUTH_JWT_SECRET=replace-with-a-long-random-secret
   AUTH_INITIAL_USERNAME=admin
   AUTH_INITIAL_PASSWORD=change-this-password
   ```

3. Start the production stack:

   ```bash
   docker compose --env-file .env.production -f docker-compose.prod.yml up --build -d
   ```

4. Open the app:

   - Frontend UI: `http://localhost:8080`
   - API through reverse proxy: `http://localhost:8080/api/v1/...`
   - Health check through reverse proxy: `http://localhost:8080/health`

To use a different public port, change `APP_PORT` in `.env.production`.

To stop the production stack:

```bash
docker compose --env-file .env.production -f docker-compose.prod.yml down
```

### Option C — Native local development

#### Run backend and frontend together

```bash
chmod +x run.sh
./run.sh
```

Default URLs:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:3000`

#### Backend only

Build the backend:

```bash
./scripts/install.sh
```

Then run:

```bash
./olt-api
```

Alternative Make targets:

```bash
make build
make run
make dev
make test
```

#### Frontend only

```bash
cd frontend
cp .env.example .env
npm install
npm run dev -- --host 0.0.0.0 --port 5173
```

## Configuration

The backend can read settings from:

1. `configs/config.yaml`
2. environment variables

Important environment variables:

```env
SERVER_HOST=0.0.0.0
SERVER_PORT=3000
DATABASE_PATH=./olt-api.db
LOGGING_LEVEL=info
LOGGING_FILE=./logs/app.log
AUTH_JWT_SECRET=replace-with-a-long-random-secret
AUTH_INITIAL_USERNAME=admin
AUTH_INITIAL_PASSWORD=admin
VITE_API_BASE_URL=http://localhost:3000
```

If `AUTH_JWT_SECRET` is not set, the backend generates a temporary secret at startup. That is acceptable for local development but not recommended for shared or production environments.

For the production stack, prefer `.env.production` and run Compose with `--env-file .env.production` so secrets and deployment-specific ports do not leak into your local development setup.

## Authentication

Default local credentials:

- Username: `admin`
- Password: `admin`

Change them through environment variables before first startup if needed:

```env
AUTH_INITIAL_USERNAME=your-admin-user
AUTH_INITIAL_PASSWORD=your-admin-password
```

## API documentation

Detailed endpoint documentation is available in [API.md](./API.md).

## Troubleshooting

### `vite: not found` or frontend dependencies look broken

Reinstall frontend dependencies:

```bash
cd frontend
rm -rf node_modules
npm install
```

For Docker Compose:

```bash
docker compose down -v
docker compose up --build
```

### SQLite or CGO build issues

The backend uses `github.com/mattn/go-sqlite3`, which requires CGO.
For native builds on Linux, install a compiler toolchain such as `gcc` / `build-essential`.

### Frontend cannot reach the backend

Check that:

1. the backend is running,
2. `SERVER_PORT` matches the published backend port,
3. `VITE_API_BASE_URL` points to the correct backend URL.

## Notes

- `docker-compose.yml` is intended for **local development**.
- `docker-compose.prod.yml` is the production-oriented full-stack setup for Docker users.
- In the production stack, the browser talks to the frontend on one port and nginx proxies API requests to the backend.
- The SQLite database and log files should not be committed.
