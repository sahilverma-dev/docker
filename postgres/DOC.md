# Docker Postgres

1. Using `docker exec` to run commands inside the container
2. Using `psql` to connect from your host machine

Here are the common commands you'll need:

1. **Connect to the container's bash:**

```bash
docker exec -it postgres bash
```

2. **Connect to PostgreSQL inside the container:**

```bash
# Once inside the container
psql -U postgres

# Or directly from your host machine
docker exec -it postgres psql -U postgres
```

3. **Create a new database:**

```sql
-- Within psql
CREATE DATABASE your_new_database;

-- Or as a one-line command from host
docker exec -it postgres psql -U postgres -c "CREATE DATABASE your_new_database;"
```

4. **List all databases:**

```sql
-- Within psql
\l
-- Or
SELECT datname FROM pg_database;

-- From host
docker exec -it postgres psql -U postgres -c "\l"
```

5. **Switch databases:**

```sql
-- Within psql
\c database_name
```

6. **Delete a database:**

```sql
-- Within psql
DROP DATABASE database_name;

-- From host
docker exec -it postgres psql -U postgres -c "DROP DATABASE database_name;"
```

Some useful aliases you can add to your `.bashrc` or `.zshrc` to make this easier:

```bash
# Add these to your shell configuration file
alias pg-connect='docker exec -it postgres psql -U postgres'
alias pg-list='docker exec -it postgres psql -U postgres -c "\l"'
alias pg-create='docker exec -it postgres psql -U postgres -c "CREATE DATABASE'
# Usage: pg-create database_name;"
```

And here's how to connect to a specific database from your applications:

- Connection string format:

```
postgresql://postgres:postgres_password@localhost:5432/your_database_name
```

For example, if you're using Python with `psycopg2`:

```python
import psycopg2

# Connect to a specific database
conn = psycopg2.connect(
    dbname="your_database_name",
    user="postgres",
    password="postgres_password",
    host="localhost",
    port="5432"
)
```

Remember:

- Default username is `postgres`
- Password is what you set in the docker-compose file
- Host is `localhost` when connecting from your machine, or `postgres` when connecting from another container
- Port is `5432`

Would you like me to provide more specific examples for any particular programming language or framework?
