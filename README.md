# Docker Starter Kit

A production-ready **Laravel 12 + React 19 + Inertia v2** starter kit, fully Dockerized with [ServerSideUp PHP](https://serversideup.net/open-source/docker-php/) images.

## Stack

| Service | Image / Tech | Port |
|---|---|---|
| **App** (Nginx + PHP-FPM) | `serversideup/php:8.5-fpm-nginx` | `80` |
| **Horizon** (Queue Manager) | `serversideup/php:8.5-fpm-nginx` | - |
| **Scheduler** (Cron) | `serversideup/php:8.5-fpm-nginx` | - |
| **Reverb** (WebSocket) | `serversideup/php:8.5-fpm-nginx` | `9001` |
| **PostgreSQL 18** | `postgres:18-alpine` | `5432` |
| **Redis** | `redis:alpine` | `6379` |
| **MinIO** (S3 Storage) | `minio/minio` | `9000` / `8900` |
| **Mailpit** (Email Testing) | `axllent/mailpit` | `8025` / `1025` |

### Frontend & Backend

- **Laravel 12** with Fortify (headless auth)
- **React 19** with Inertia v2
- **Tailwind CSS v4**
- **Vite 7** with HMR
- **Laravel Horizon** for queue management & dashboard
- **Laravel Reverb** for real-time WebSocket broadcasting
- **Laravel Wayfinder** for type-safe route generation
- **Pest 4** for testing

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + Compose)
- [Node.js](https://nodejs.org/) >= 20
- [Composer](https://getcomposer.org/)

## Getting Started

**1. Clone the repository:**

```bash
git clone https://github.com/baconfy/docker-starter-kit.git
cd docker-starter-kit
```

**2. Install dependencies:**

```bash
composer install
npm install
```

**3. Set up environment:**

```bash
cp .env.example .env
php artisan key:generate
```

**4. Start everything:**

```bash
composer dev
```

This single command will:

- Start all Docker containers (PostgreSQL, Redis, MinIO, Mailpit, etc.)
- Wait for all services to be healthy
- Create the MinIO storage bucket
- Run migrations and seed the database
- Start Vite dev server with HMR

**5. Open the app:**

- **App:** [http://localhost](http://localhost)
- **Horizon Dashboard:** [http://localhost/horizon](http://localhost/horizon)
- **Mailpit:** [http://localhost:8025](http://localhost:8025)
- **MinIO Console:** [http://localhost:8900](http://localhost:8900)

### Default Credentials

| Service | Username | Password |
|---|---|---|
| **App** (seeded user) | `root@app.com` | `password` |
| **PostgreSQL** | `baconfy` | `secret` |
| **MinIO** | `baconfy` | `secret123` |

## Commands

| Command | Description |
|---|---|
| `composer dev` | Start all services + Vite (fresh database each time) |
| `composer dev:stop` | Stop all services |
| `composer dev:fresh` | Same as `dev` with `migrate:fresh --seed` |
| `composer test` | Run lint + tests |
| `composer lint` | Fix code style with Pint |

## Port Customization

All ports are configurable via environment variables to avoid conflicts:

```env
FORWARD_APP_PORT=80
FORWARD_DB_PORT=5432
FORWARD_REDIS_PORT=6379
FORWARD_REVERB_PORT=9001
FORWARD_MINIO_PORT=9000
FORWARD_MINIO_CONSOLE_PORT=8900
FORWARD_MAILPIT_PORT=8025
FORWARD_MAILPIT_SMTP_PORT=1025
```

## Architecture

The project uses a **two-file Docker Compose pattern**:

- **`docker-compose.yml`** — Production base with multi-stage Dockerfile build, OPcache enabled, healthchecks, and auto-migrations.
- **`docker-compose.dev.yml`** — Dev overrides that mount your local code, disable OPcache, expose database/Redis ports, and add Mailpit.

### Dockerfile (Multi-Stage)

```
Stage 1: node:22-alpine        → Build frontend assets (Vite)
Stage 2: serversideup/php:8.5-cli → Install Composer dependencies
Stage 3: serversideup/php:8.5-fpm-nginx → Production image
```

### Healthchecks

All services have healthchecks configured. The `composer dev` command uses `--wait` to ensure everything is healthy before running migrations.

## Production

Build and deploy with the production compose file:

```bash
docker compose up -d --build --wait
```

The production image includes:

- OPcache enabled
- Auto-migrations on startup
- Auto storage link creation
- Optimized Composer autoloader
- Pre-built frontend assets

## License

[MIT](LICENSE)
