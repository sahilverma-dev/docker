# Redis + RedisInsight Docker Setup

A Docker Compose setup for running **Redis 7** with persistent storage, password authentication, health checks, resource limits, and **RedisInsight** for database management.

## Features

* Redis 7 running on Alpine Linux
* Password-protected Redis instance
* Persistent Redis data using Docker volumes
* AOF (Append Only File) enabled
* RDB snapshots enabled
* Redis health check
* CPU and memory limits
* RedisInsight web UI
* Isolated Docker bridge network
* Environment-variable-based port configuration

## Project Structure

```text
.
├── docker-compose.yml
└── .env
```

## Docker Compose

The setup contains two services:

| Service        | Description             | Default Port |
| -------------- | ----------------------- | -----------: |
| `redis`        | Redis database          |       `6379` |
| `redisinsight` | Redis web management UI |       `8001` |

Both services communicate through the `redis-net` Docker network.

## Environment Variables

Create a `.env` file in the same directory as `docker-compose.yml`:

```env
REDIS_PORT=6379
REDIS_PASSWORD=password123
REDISINSIGHT_PORT=8001
```

The following defaults are also configured in `docker-compose.yml`:

```text
REDIS_PORT        → 6379
REDIS_PASSWORD    → password123
REDISINSIGHT_PORT → 8001
```

> **Security:** Change `REDIS_PASSWORD` before using this setup outside a local development environment.

## Start the Services

Start Redis and RedisInsight in detached mode:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

View Redis logs:

```bash
docker compose logs -f redis
```

View RedisInsight logs:

```bash
docker compose logs -f redisinsight
```

## Redis Health Check

The Redis container includes a health check using `redis-cli`:

```bash
redis-cli -a "$REDIS_PASSWORD" ping
```

A healthy Redis instance should return:

```text
PONG
```

RedisInsight starts after Redis has passed its health check.

## RedisInsight

Once the containers are running, open:

```text
http://localhost:8001
```

When adding the Redis database in RedisInsight, use:

```text
Host: redis
Port: 6379
Username: default
Password: password123
```

Because RedisInsight is running inside the same Docker network, it should connect to Redis using the service name `redis`, rather than `localhost`.

If you are connecting to RedisInsight from another machine, access it using the Docker host's IP address:

```text
http://<SERVER_IP>:8001
```

## Connecting from TypeScript

Install `ioredis`:

```bash
npm install ioredis
```

Then create a Redis connection:

```typescript
import Redis from "ioredis";

const redis = new Redis({
  host: "192.168.1.43",
  port: 6379,
  password: "password123",
});

const data = await redis.get("data");

console.log(data);
```

### Using Environment Variables

It is recommended not to hard-code the Redis password in your TypeScript application.

Install the environment variable support if needed:

```bash
npm install dotenv
```

Create a `.env` file:

```env
REDIS_HOST=192.168.1.43
REDIS_PORT=6379
REDIS_PASSWORD=password123
```

Then:

```typescript
import "dotenv/config";
import Redis from "ioredis";

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: Number(process.env.REDIS_PORT ?? 6379),
  password: process.env.REDIS_PASSWORD,
});

const data = await redis.get("data");

console.log(data);
```

## Setting and Getting Data

Set a value:

```typescript
await redis.set("data", "Hello Redis");
```

Get the value:

```typescript
const data = await redis.get("data");

console.log(data);
```

Output:

```text
Hello Redis
```

## Connecting from Another Docker Container

If your TypeScript application also runs inside Docker and is connected to `redis-net`, use the Redis service name:

```typescript
const redis = new Redis({
  host: "redis",
  port: 6379,
  password: process.env.REDIS_PASSWORD,
});
```

Do **not** use:

```typescript
host: "localhost"
```

because `localhost` inside your application container refers to the application container itself.

## Data Persistence

Redis uses two persistence mechanisms in this configuration.

### RDB Snapshots

```text
--save 60 1
```

Redis creates a snapshot when at least one key has changed within 60 seconds.

### AOF

```text
--appendonly yes
```

Redis also records write operations using Append Only File persistence.

The Redis data is stored in the Docker volume:

```text
redis_data:/data
```

The data therefore survives container restarts and recreation as long as the Docker volume is retained.

## Volumes

Two named volumes are created:

```text
redis_data
```

Stores Redis database data.

```text
redisinsight_data
```

Stores RedisInsight application data and configuration.

List volumes:

```bash
docker volume ls
```

## Network

Both services use:

```text
redis-net
```

This allows RedisInsight and other containers connected to the same network to communicate with Redis using:

```text
redis:6379
```

## Stop the Services

Stop the containers:

```bash
docker compose stop
```

Remove the containers:

```bash
docker compose down
```

## Remove Containers and Data

To remove the containers **and their persistent volumes**:

```bash
docker compose down -v
```

> **Warning:** `docker compose down -v` permanently removes the Redis and RedisInsight Docker volumes. Any data stored in those volumes will be lost.

## Useful Commands

### Check Redis directly

```bash
docker exec -it redis redis-cli -a password123 ping
```

### Open Redis CLI

```bash
docker exec -it redis redis-cli -a password123
```

Then:

```redis
SET data "Hello Redis"
GET data
```

### Check Redis keys

```redis
KEYS *
```

For production systems with many keys, prefer `SCAN` instead of `KEYS *`.

### Restart Redis

```bash
docker compose restart redis
```

### Recreate the setup

```bash
docker compose down
docker compose up -d
```

## Security Considerations

This configuration is suitable for local development and internal networks, but should be hardened before production use.

* Use a strong, unique `REDIS_PASSWORD`.
* Avoid committing `.env` files containing credentials.
* Restrict access to port `6379` using a firewall.
* Do not expose Redis directly to the public internet.
* Consider TLS when Redis traffic crosses an untrusted network.
* Use a reverse proxy and authentication if exposing RedisInsight externally.
* Keep the Redis and RedisInsight images updated.

Add `.env` to `.gitignore`:

```gitignore
.env
```

## Troubleshooting

### Redis is unhealthy

Check the logs:

```bash
docker compose logs redis
```

Test Redis manually:

```bash
docker exec -it redis redis-cli -a password123 ping
```

Expected result:

```text
PONG
```

### TypeScript cannot connect

Verify that Redis is listening on the expected port:

```bash
docker compose ps
```

If your application runs outside Docker, make sure the Docker host IP is correct:

```typescript
host: "192.168.1.43"
```

Also make sure port `6379` is reachable from the machine running your TypeScript application.

### RedisInsight cannot connect to Redis

When RedisInsight is running in Docker, use:

```text
Host: redis
Port: 6379
Password: password123
```

Do not use the host machine's LAN IP unless you specifically need to connect that way.

## License

Use and modify this configuration as needed for your project.
