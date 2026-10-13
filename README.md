# test-manager-library-3145556-infra

> **Prueba técnica · Ficha ADSO 3145556** — Infraestructura de *Manager Library*: contenedores, red, volúmenes y motor de base de
> datos vacío. Spec: `test-manager-library-3145556-docs` (`spec-python/` y la traducción en `spec-laravel/`).

## Qué levanta

| Servicio | Imagen / build | Puerto | Papel |
|---|---|---|---|
| `db` | `postgres:16` | 5432 | Motor **vacío**: solo base, usuario y volumen |
| `api` | `../test-manager-library-3145556-api` | 8000 | Laravel; al arrancar corre las migraciones y crea el admin inicial |
| `app` | `../test-manager-library-3145556-app` | 5173 | React servido como estático |

Red `library-net`; volúmenes `db-data` (datos) y `covers` (portadas).

**Este repositorio no contiene DDL (creación de tablas) ni scripts en `docker-entrypoint-initdb.d`** (Art. V, ADR-001). Todo el DDL vive en
las migraciones de la API.

## Cómo se usa (un solo comando)

Los seis repositorios deben estar clonados uno al lado del otro:

```text
PruebaFinal/
├── test-manager-library-3145556-api/
├── test-manager-library-3145556-app/
└── test-manager-library-3145556-infra/   ← aquí
```

```bash
cp .env.example .env        # completar POSTGRES_PASSWORD, APP_KEY, JWT_SECRET, ADMIN_PASSWORD
docker compose up -d --build
```

## Cómo se verifica (evidencia de T-20 y RNF-5)

```bash
# 1. La API responde
curl http://localhost:8000/api/v1/health          # {"status":"ok"}

# 2. Las tablas existen solo porque las creó la migración de la API
docker compose exec db psql -U library -d library -c "\dt library.*"

# 3. Este repo no crea tablas
grep -riE "create[[:space:]]+table" --exclude-dir=.git . || echo "sin DDL en infra"

# 4. Iniciar sesión con el admin del .env
curl -s -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"<ADMIN_PASSWORD>"}'
```

Para comprobar que la base arranca vacía: `docker compose down -v && docker compose up -d db`, y antes de levantar la API,
`docker compose exec db psql -U library -d library -c "\dn"` no muestra el esquema `library`.

## Fase de demostración con el reloj desplazado (ADR-006)

Solo para el sembrador de `tool`: `CLOCK_OFFSET_DAYS=-20 docker compose up -d api`, correr la fase `past` del sembrador y volver a
`CLOCK_OFFSET_DAYS=0 docker compose up -d api`.
