This project shows how to deploy simple web application using Docker Compose and Kubernetes.


## Prerequisites

- **Docker** — required to run the stack. Install Docker Engine and Docker Compose:
  - [Install Docker](https://docs.docker.com/get-docker/)

## Docker Compose

This repo includes a Docker Compose manifest at `docker/docker-compose.yml`. It runs three services:

| Service    | Description                                      | Port  |
|-----------|---------------------------------------------------|-------|
| **postgres** | PostgreSQL 17.4 database (`notesdb`)             | 5432  |
| **notes-api** | Notes REST API (Spring Boot)                     | 8080  |
| **notes-ui**  | Notes web frontend                               | 5173  |

## How to launch

Go to the docker directory from the repo root:
```bash
cd docker
```

Run the stack in the background (logs are not shown in the terminal):
```bash
docker compose up -d
```

To run in the foreground (logs in terminal):

```bash
docker compose up
```

To stop and remove containers:

```bash
docker compose down
```

## Accessing the frontend

Open a browser and go to:

**http://localhost:5173**

The UI talks to the API at `http://localhost:8080`.

Default user credentials

- `gleb` / `gleb123`
- `test` / `test123`