# MobilityTicketing: SQL programmability lab

This is the implementation lab for the third databases lecture. It uses the same small MobilityTicketing case and compares four ways to produce daily captured revenue:

1. a direct aggregate query;
2. a SQL function over the base tables;
3. a materialized view;
4. a trigger-maintained summary table.

The supplied trigger is intentionally incomplete. The lab is successful when the differences are observable, explained, and tied to a recommendation.

## Requirements

- Docker Desktop with Compose
- The lecture notes and the implementation lab in [`docs/lab.md`](docs/lab.md)

## Start the database

```bash
docker compose up -d
docker compose ps
```

The database is available at `localhost:5432` with database `mobility`, user `mobility`, and password `mobility`.

The initialisation scripts load the relational and ticketing data. They run when the PostgreSQL container is created. To start again from the seeded state:

```bash
docker compose down
docker compose up -d
```

Apply a migration from the repository root with:

```bash
docker compose exec -T postgres psql -U mobility -d mobility < database/postgres/migrations/020_reporting_function.sql
```

## Materials

- `compose.yaml`: local PostgreSQL infrastructure.
- `database/postgres/init/`: relational and ticketing starter schema plus seed data.
- `database/postgres/migrations/`: incomplete student migration examples.
- `database/postgres/queries/`: released reporting query.
- `database/postgres/experiments/`: test cases for inserts, corrections, deletes, and duplicate delivery.
- `docs/`: the lab