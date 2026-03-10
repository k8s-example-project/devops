## Kubernetes (Minikube) setup

This folder contains Kubernetes manifests to run the **notes** application (UI + API + Postgres) locally on **Minikube**.

### Prerequisites

- **Docker**: required if you use the Docker driver for Minikube.
- **Minikube**: install from the official docs at `https://kubernetes.io/ru/docs/tasks/tools/install-minikube/`
- **kubectl**: install from the official docs at `https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/`
- **Local hosts file access**: to add `notes.host -> 127.0.0.1` (admin rights may be required on Windows).

### Install / start Minikube

Start Minikube (Docker driver) with resources appropriate for the stack:

```bash
minikube start --driver=docker --cpus=4 --memory=16g
```

Verify it’s running:

```bash
minikube status
kubectl version --client
```

Enable the NGINX Ingress controller addon used by `project/ingress.yaml`:

```bash
minikube addons enable ingress
```

Optional: open the Kubernetes dashboard for monitoring with GUI:

```bash
minikube dashboard
```

### Apply manifests (deploy)

Create the namespace (all manifests are pinned to `namespace: notes`):

```bash
kubectl create ns notes
```

Apply **all** manifests from `k8s/project/` recursively, it can take some time:

```bash
kubectl apply -f k8s/project/ -R
```

Check rollout / pods:

```bash
kubectl get pods -n notes
kubectl get svc -n notes
kubectl get ingress -n notes
```

### Access the application locally

The Ingress is configured for host **`notes.host`**.

1) Add a hosts entry:

- **Windows**: edit `C:\Windows\System32\drivers\etc\hosts` (run your editor as Administrator)
- Add the line:

```text
127.0.0.1 notes.host
```

2) Run Minikube tunnel (keep this terminal running):

```bash
minikube tunnel
```

3) Open in the browser:

- UI: `http://notes.host/`

### Scaling the application manually (optional)

To see how scaling works, you can manually scale the application:

```bash
kubectl scale deployment notes-ui -n notes --replicas=3
kubectl scale deployment notes-api -n notes --replicas=3
```

To see the current amount of pods:

```bash
kubectl get pods -n notes
```

To scale back to 1 replica:

```bash
kubectl scale deployment notes-ui -n notes --replicas=1
kubectl scale deployment notes-api -n notes --replicas=1
```

### Files in `k8s/`

```text
k8s/
├── README.MD                    # Kubernetes / Minikube setup guide
└── project/
    ├── ingress.yaml             # NGINX Ingress: notes.host, /api -> notes-api:8080, / -> notes-ui:80
    ├── notes-api/
    │   ├── deployment.yaml      # notes-api Deployment (Spring), exposes 8080, reads DB creds from Secret/ConfigMap
    │   ├── service.yaml         # ClusterIP Service for notes-api (port 8080)
    │   └── configmap.yaml       # SPRING_DATASOURCE_URL (jdbc:postgresql://postgres.notes.svc.cluster.local:5432/notesdb)
    ├── notes-ui/
    │   ├── deployment.yaml      # notes-ui Deployment (Vite UI), exposes 80, reads VITE_API_BASE_URL
    │   ├── service.yaml         # ClusterIP Service for notes-ui (port 80)
    │   └── configmap.yaml       # VITE_API_BASE_URL (set to "/")
    └── postgres/
        ├── statefulset.yaml     # Postgres StatefulSet with PVC (10Gi) at /var/lib/postgresql/data
        ├── service.yaml         # Headless Service (clusterIP: None) for stable DNS `postgres`
        └── secret.yaml          # Demo Secret: POSTGRES_DB/USER/PASSWORD used by Postgres and notes-api
```