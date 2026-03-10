## Notes application – Docker & Kubernetes example

This project demonstrates how to run a simple **notes** web application (Postgres + Spring Boot API + web UI) using **Docker Compose** and **Kubernetes (Minikube)**.

The same application stack is provided in two flavours:

- **Docker Compose** – quick local setup for development
- **Kubernetes / Minikube** – more production‑like environment with Ingress, StatefulSet, and ConfigMaps/Secrets

### Where to start

- **Docker setup**: see `docker/README.md` for running the stack with Docker Compose.
- **Kubernetes / Minikube setup**: see `k8s/README.MD` for deploying the stack to Minikube.

### High‑level components

- **postgres**: PostgreSQL database (`notesdb`), stores notes and user data.
- **notes-api**: Spring Boot REST API providing CRUD endpoints for notes and authentication.
- **notes-ui**: Web frontend for interacting with the notes application.

## Default user credentials

- `gleb` / `gleb123`
- `test` / `test123`