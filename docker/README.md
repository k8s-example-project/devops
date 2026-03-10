## Docker Compose setup

This directory contains a Docker Compose setup for running the **notes** application locally (Postgres + API + UI) without Kubernetes.

### Prerequisites

- **Docker** and **Docker Compose** (or `docker compose` CLI)
  - Install Docker: `https://docs.docker.com/get-docker/`

### Stack overview

Compose file: `docker-compose.yml`

Services:

| Service       | Description                                      | Port  |
|--------------|--------------------------------------------------|-------|
| **postgres**  | PostgreSQL 17.4 database (`notesdb`)             | 5432  |
| **notes-api** | Notes REST API (Spring Boot)                     | 8080  |
| **notes-ui**  | Notes web frontend                               | 5173  |

### How to launch

From the repo root, go to the `docker` directory:

```bash
cd docker
```

Run the stack in the background (containers start, logs stay in the background):

```bash
docker compose up -d
```

Run in the foreground (logs printed in the current terminal):

```bash
docker compose up
```

Stop and remove containers:

```bash
docker compose down
```

### Accessing the application

Open the frontend:

```text
http://localhost:5173
```

The UI talks to the API at:

```text
http://localhost:8080
```

Default user credentials:

- `gleb` / `gleb123`
- `test` / `test123`

