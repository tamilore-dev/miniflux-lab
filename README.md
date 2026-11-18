# miniflux-lab

A small Docker Compose lab: the [Miniflux](https://miniflux.app) RSS reader backed by PostgreSQL. I built it to learn how Docker networking works between containers.

## Architecture

```
browser ──127.0.0.1:8080──► miniflux ──"db":5432──► postgres
```

- Two containers on one private network called `backend`.
- Only Miniflux is published to the host. Postgres has no published port, so nothing outside the network can reach it.
- Miniflux finds the database by its service name (`db`) using Docker's built-in DNS.
- Database files live in a named volume, so data survives container removal.

## What this demonstrates

- Service discovery by name on a user-defined bridge network
- Published vs unpublished ports (`ports:` on one service, none on the other)
- Binding a port to `127.0.0.1` so it is not exposed to the local network
- Healthchecks plus `depends_on: condition: service_healthy` for startup order
- Named volumes for persistent data
- Keeping secrets out of Git with `.env` and `.env.example`

## Requirements

- Docker with the Compose plugin (`docker compose version` should print a version)

## Quick start

```bash
git clone <your-repo-url>
cd miniflux-lab
cp .env.example .env
nano .env                # set real passwords
docker compose up -d
docker compose ps        # db should show (healthy)
```

Open http://localhost:8080 and log in as `admin` with the `ADMIN_PASSWORD` from your `.env`.

## Configuration

| Variable | Purpose |
|---|---|
| `POSTGRES_PASSWORD` | Password for the Postgres user `miniflux` |
| `ADMIN_PASSWORD` | Password for the Miniflux `admin` login |

`.env` is ignored by Git. Never commit it.

> Postgres only reads `POSTGRES_PASSWORD` the first time it creates the volume. If you change it later, either keep the old value or reset with
