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

### Apply manifests manually (deploy)

This is a manual way to deploy the application, you can also use Argo CD to deploy the application (see below).

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

### Argo CD deployment (GitOps)

You can also manage these Kubernetes manifests using **Argo CD**.

- **Install Argo CD**:

  ```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

- **Check that Argo CD is running**:

  ```bash
  kubectl get pods -n argocd
  ```

- **Port‑forward Argo CD UI**:

  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8234:443
  ```

  Open the UI in a browser at:

  ```text
  https://localhost:8234
  ```

- **Log in to the Argo CD UI**:

  - Username: `admin`
  - Password (initial, from secret):

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode; echo
    ```

- **Connect this GitHub repo (if needed)**:

  This project uses the public repository:

  ```text
  https://github.com/k8s-example-project/devops.git
  ```

  For public read‑only access you usually don’t need credentials. If you want to use HTTPS with credentials (e.g. for a private fork):

  - Go to `Settings / Repositories` in Argo CD UI.
  - Click `CONNECT REPO`.
  - Set:
    - Connection method: `VIA HTTPS`
    - Type: `git`
    - Repository URL: `https://github.com/k8s-example-project/devops.git`
    - Username / Password: your GitHub username and a personal access token (for private repos).

- **Apply the Argo CD Application for this cluster**:


  The manifest `k8s/cluster-apps.yaml` creates an Argo CD `Application` that points at `k8s/argo-apps`, which in turn defines the `notes` and `postgres` Applications.

  ```bash
  kubectl apply -f k8s/cluster-apps.yaml
  ```

  - OR manually create application in Argo CD UI with link to this manifest
  - Go to ```Apllications```
  - Click on ```NEW APP```
  - Fill in data about app:
    - Application name: ```my-cluster-apps```
    - Project name: ```default```
    - Repository URL: ```https://github.com/k8s-example-project/devops.git```
    - Revision: ```HEAD```
    - Path: ```./k8s/```
    - Cluster URL: ```https://kubernetes.default.svc```
    - Namespace: ```default```

  After this:

  - Argo CD will manage the `notes` and `postgres` Applications defined in `k8s/argo-apps/`.
  - Auto‑sync (with `selfHeal` and `prune`) is enabled for `cluster-apps` and for the child Applications, so changes pushed to the Git repo are automatically applied to the cluster.

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



### Files in `k8s/`

```text
k8s/
├── README.MD                    # Kubernetes / Minikube + Argo CD setup guide
├── cluster-apps.yaml            # Argo CD Application that manages apps from k8s/argo-apps
├── argo-apps/
│   ├── argocd-application-sam-mcp.yaml    # Argo CD Application for notes UI/API + ingress
│   └── argocd-application-postgres.yaml   # Argo CD Application for postgres in the target namespace
└── project/
    ├── ingress.yaml             # NGINX Ingress: notes.host, /api -> notes-api:8080, / -> notes-ui:80
    ├── notes-api/
    │   ├── deployment.yaml      # notes-api Deployment (Spring), exposes 8080, reads DB creds from Secret/ConfigMap
    │   ├── service.yaml         # ClusterIP Service for notes-api (port 8080)
    │   └── configmap.yaml       # SPRING_DATASOURCE_URL (jdbc:postgresql://postgres.notes.svc.cluster.local:5432/notesdb)
    ├── notes-ui/
    │   ├── deployment.yaml      # notes-ui Deployment (Vue 3), exposes 80, reads VITE_API_BASE_URL
    │   ├── service.yaml         # ClusterIP Service for notes-ui (port 80)
    │   └── configmap.yaml       # VITE_API_BASE_URL (set to "/")
    └── postgres/
        ├── statefulset.yaml     # Postgres StatefulSet with PVC (10Gi) at /var/lib/postgresql/data
        ├── service.yaml         # Headless Service (clusterIP: None) for stable DNS `postgres`
        └── secret.yaml          # Demo Secret: POSTGRES_DB/USER/PASSWORD used by Postgres and notes-api
```