# Velum — All-in-One Docker Image

This image packages **PostgreSQL + backend + frontend** into a single container. Your friends do **not** need the source code.

## Quick Start (no build)

### Option 1: `docker run` (one-liner)

```bash
docker run -d \
  -p 3000:80 \
  -v velum-data:/var/lib/postgresql/data \
  -v velum-secrets:/var/lib/budget-secrets \
  --name velum \
  petergelgor/velum:latest
```

Open **http://localhost:3000**.

### Option 2: `docker compose` (also no build)

Create a `docker-compose.yml` like this:

```yaml
services:
  velum:
    image: petergelgor/velum:latest
    container_name: velum
    restart: unless-stopped
    ports:
      - "3000:80"
    volumes:
      - velum_data:/var/lib/postgresql/data
      - velum_secrets:/var/lib/budget-secrets

volumes:
  velum_data:
  velum_secrets:
```

Then run:

```bash
docker compose up -d
```

### Stop / restart / remove

```bash
docker stop velum
docker start velum

docker rm -f velum
docker volume rm velum_data velum_secrets   # deletes all data
```

## Configuration (optional)

You can customize behavior with environment variables:

| Variable            | Default   | Description                        |
|---------------------|-----------|------------------------------------|
| `POSTGRES_DB`       | `budget`  | Database name                      |
| `POSTGRES_USER`     | `budget`  | Database username                  |
| `POSTGRES_PASSWORD` | `budget`  | Database password                  |
| `JWT_SECRET`        | (auto)    | Secret for signing auth tokens     |

Example with custom settings:

```bash
docker run -d -p 3000:80 \
  -e JWT_SECRET="my-super-secret-key-at-least-32-bytes-long" \
  -v velum-data:/var/lib/postgresql/data \
  -v velum-secrets:/var/lib/budget-secrets \
  --name velum \
  petergelgor/velum:latest
```

## Developers: build locally (optional)

From the repo root:

```bash
docker build -t velum -f all-in-one/Dockerfile .
```
