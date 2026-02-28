# Local PostgreSQL Dev Container

A simple Docker setup providing a reusable local PostgreSQL database + pgAdmin UI for development across projects.

---

## Services

- **Postgres 15** – primary database
- **pgAdmin 4** – browser UI to inspect and manage DB

Ports:

- Postgres → `localhost:5432`
- pgAdmin → `http://localhost:5050`

Default credentials:

| Item     | Value |
| -------- | ----- |
| User     | dev   |
| Password | dev   |
| Database | app   |

Connection URL:

```bash
postgresql://dev:dev@localhost:5432/app
```

---

## Start Database

```bash
docker compose up -d
```

Stop (keep data):

```bash
docker compose down
```

Reset / wipe database completely:

```bash
docker compose down -v
```

---

## pgAdmin Login

Open browser:

```txt
http://localhost:5050
```

Login:

| Field    | Value                                 |
| -------- | ------------------------------------- |
| Email    | [dev@local.com](mailto:dev@local.com) |
| Password | dev                                   |

---

## Add Database Server in pgAdmin

Important: Do NOT use localhost

Docker containers communicate using service names.

### General Tab

Name:

```txt
Local Postgres
```

### Connection Tab

| Field          | Value    |
| -------------- | -------- |
| Host name      | postgres |
| Port           | 5432     |
| Maintenance DB | app      |
| Username       | dev      |
| Password       | dev      |
| Save Password  | ON       |

---

## Using From Applications

Works with:

- Prisma
- Drizzle
- Sequelize
- TypeORM
- NestJS
- Next.js
- Bun
- Node.js

Use this environment variable in projects:

```env
DATABASE_URL=postgresql://dev:dev@localhost:5432/app
```

---

## Troubleshooting

### 1) Connection refused

Postgres not ready yet.

Check:

```bash
docker logs local_postgres
```

Wait for:

```bash
database system is ready to accept connections
```

---

### 2) pgAdmin cannot connect using localhost

Inside Docker, `localhost` means the pgAdmin container itself.

Use:

```bash
postgres
```

NOT `localhost` or `127.0.0.1`

---

### 3) Password authentication failed

Postgres only reads credentials on first startup.
If credentials changed later, the volume still contains old users.

Fix:

```bash
docker compose down -v
docker compose up -d
```

---

### 4) fe_sendauth: no password supplied

pgAdmin didn’t store password.
Edit server → Connection → enable **Save Password**.

---

### 5) Verify DB is reachable from host

```bash
psql postgresql://dev:dev@localhost:5432/app
```

---

## Notes

- Data persists between restarts
- Safe to reuse across all local projects
- Delete volume only when you want a fresh DB
- Designed for local development only (not production)
