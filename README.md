# test-manager-library-3145556-infra

> **Technical test · ADSO group 3145556**: *Manager Library* infrastructure: containers, network, volumes and an empty
> database engine. Spec: `test-manager-library-3145556-docs` (`spec-python/` and its translation in `spec-laravel/`).

## What it starts

| Service | Image / build | Port | Role |
|---|---|---|---|
| `db` | `postgres:16` | 5432 | **Empty** engine: only the database, the user and the volume |
| `api` | `../test-manager-library-3145556-api` | 8000 | Laravel. At startup it runs the migrations and creates the initial admin |
| `app` | `../test-manager-library-3145556-app` | 5173 | React served as static files |

Network `library-net`; volumes `db-data` (data) and `covers` (covers).

**This repository holds no DDL (table creation) and no scripts in `docker-entrypoint-initdb.d`** (Art. V, ADR-001). All the DDL
lives in the API migrations.

## How to use it (one command)

The six repositories must be cloned next to each other:

```text
PruebaFinal/
├── test-manager-library-3145556-api/
├── test-manager-library-3145556-app/
└── test-manager-library-3145556-infra/   ← here
```

```bash
cp .env.example .env        # fill in POSTGRES_PASSWORD, APP_KEY, JWT_SECRET and ADMIN_PASSWORD
docker compose up -d --build
```

The admin username defaults to `Jonas7891`. Passwords go only in the local `.env`, never in the repository (ADR-014).

## How to verify it (T-20 and RNF-5 evidence)

```bash
# 1. The API answers
curl http://localhost:8000/api/v1/health          # {"status":"ok"}

# 2. The tables exist only because the API migration created them
docker compose exec db psql -U library -d library -c "\dt library.*"

# 3. This repo creates no tables
grep -riE "create[[:space:]]+table" --exclude-dir=.git . || echo "no DDL in infra"

# 4. Log in with the admin from .env
curl -s -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"Jonas7891","password":"<ADMIN_PASSWORD>"}'
```

To check that the database starts empty: run `docker compose down -v && docker compose up -d db`. Before starting the API,
`docker compose exec db psql -U library -d library -c "\dn"` does not list the `library` schema.

## Demo phase with a shifted clock (ADR-006)

Only for the `tool` seeder:
1. `CLOCK_OFFSET_DAYS=-20 docker compose up -d api`;
2. run the seeder's `past` phase;
3. go back with `CLOCK_OFFSET_DAYS=0 docker compose up -d api`.
