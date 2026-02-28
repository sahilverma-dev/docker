# Docker Postgres (Shared Local Database Server)

This repository provides a **single reusable PostgreSQL server** for all your local projects.

You will have ONE server in pgAdmin named:

```bash
Docker
```

Inside it you can create unlimited databases:

- finance-platform
- auth-service
- analytics
- test-db
- anything else

So you never create new containers per project again.

---

## Services

| Service     | Purpose        | Port |
| ----------- | -------------- | ---- |
| Postgres 15 | Main DB server | 5432 |
| pgAdmin     | Database UI    | 5050 |

Credentials:

| Field      | Value    |
| ---------- | -------- |
| User       | dev      |
| Password   | dev      |
| Default DB | postgres |

---

## Start Database Server

```base
docker compose up -d
```

Stop server (keep data):

```base
docker compose down
```

Reset entire database server (delete ALL databases):

```base
docker compose down -v
```

---

## Access pgAdmin

Open:

```txt
http://localhost:5050
```

Login:

| Field    | Value                                 |
| -------- | ------------------------------------- |
| Email    | [dev@local.com](mailto:dev@local.com) |
| Password | dev                                   |

---

## Add Server in pgAdmin (IMPORTANT)

Create ONLY ONE server entry.

### General Tab

Name:

```base
Docker
```

### Connection Tab

| Field          | Value    |
| -------------- | -------- |
| Host name      | postgres |
| Port           | 5432     |
| Maintenance DB | postgres |
| Username       | dev      |
| Password       | dev      |
| Save Password  | ON       |

After saving — this server will contain all your project databases.

---

## Creating a New Database (Per Project)

Right click:

```base
Docker → Databases → Create → Database
```

Example:

| Project      | Database Name         |
| ------------ | --------------------- |
| Finance App  | finance-platform      |
| Auth Service | auth                  |
| SaaS App     | saas                  |
| Tests        | finance-platform-test |

---

## Connection Strings

From your local apps (Node / Bun / Prisma / Drizzle):

```base
postgresql://dev:dev@localhost:5432/<database-name>
```

Example:

```env
DATABASE_URL=postgresql://dev:dev@localhost:5432/finance-platform
```

---

## Using with Drizzle

1. Create database in pgAdmin
2. Set DATABASE_URL
3. Run:

```base
npx drizzle-kit push
```

```base
bunx drizzle-kit push
```

---

## Troubleshooting

### Cannot connect using localhost inside pgAdmin

Use:

```base
postgres
```

NOT localhost.

---

### database does not exist

Create it manually in pgAdmin first.

---

### password authentication failed

Reset server:

```base
docker compose down -v
docker compose up -d
```

---

### fe_sendauth: no password supplied

Edit server → enable Save Password.

---

### Check DB ready

```base
docker logs local_postgres
```

Wait for:

```base
database system is ready to accept connections
```
