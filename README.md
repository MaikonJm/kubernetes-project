# Kubernetes Orchestration: Deployment, Ingress & Autoscaling

> Deploys a containerized web app on a local Kubernetes cluster (Minikube/Kind) with configuration management, health checks, Ingress routing and CPU-based **Horizontal Pod Autoscaling**. Manifests are validated in CI.

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-F7B93E?logo=kubernetes&logoColor=black)
![CI](https://github.com/MaikonJm/kubernetes-project/actions/workflows/validate.yml/badge.svg)

## TL;DR

| | |
|---|---|
| **What** | Namespace, Deployment, Service, Ingress, ConfigMap, Secret and HPA for a web app (NGINX), plus an optional admin user for the Kubernetes Dashboard |
| **Stack** | Kubernetes · Minikube or Kind · NGINX Ingress Controller · metrics-server · GitHub Actions + kubeconform |
| **Result** | App scales automatically from **2 to 5 replicas** when average CPU passes 50% |

## Architecture

```text
client ──► Ingress (my-app.local) ──► Service (ClusterIP) ──► Pods (Deployment, 2–5 replicas)
                                                                  ▲
                                         HPA (CPU 50%) ───────────┘
                                         ConfigMap + Secret ──► env vars
```

### Key design decisions

- **CPU requests and limits on the container.** The HPA calculates utilization as a percentage of the *request*; without requests, autoscaling never triggers.
- **Readiness and liveness probes.** Traffic only reaches pods that are ready, and stuck pods are restarted automatically.
- **Service as `ClusterIP` + Ingress.** One entry point with host-based routing instead of a LoadBalancer per service.
- **Pinned image tag** (`nginx:1.27-alpine`) instead of `:latest`, for reproducible deployments.
- **Dedicated namespace** (`my-app`) to isolate the app's resources.
- **Configuration outside the image.** Settings come from a ConfigMap and the password from a Secret, both injected as environment variables.

## Repository structure

```text
k8s/
├── namespace.yaml        # my-app namespace
├── deployment.yaml       # app pods: resources, probes, env from ConfigMap/Secret
├── service.yaml          # ClusterIP service
├── ingress.yaml          # host-based routing (my-app.local)
├── configmap.yaml        # non-sensitive configuration
├── secrets.yaml          # example Secret (placeholder value)
├── hpa.yaml              # Horizontal Pod Autoscaler (CPU 50%, 2–5 replicas)
└── dashboard/            # optional: admin user for the Kubernetes Dashboard
    ├── admin-user.yaml
    └── admin-user-binding.yaml
```

## How to run (Minikube)

```bash
minikube start
minikube addons enable ingress
minikube addons enable metrics-server   # required by the HPA

kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/

kubectl get pods,svc,ingress,hpa -n my-app
```

Add `$(minikube ip) my-app.local` to your `/etc/hosts` and open `http://my-app.local`.
Quick alternative: `kubectl port-forward svc/my-app-service 8080:80 -n my-app`, then open `http://localhost:8080`.

### Test the autoscaling

```bash
# generate load from inside the cluster
kubectl run load -n my-app --rm -it --image=busybox:1.36 --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://my-app-service; done"

# in another terminal: watch replicas go up
kubectl get hpa -n my-app -w
```

### Optional: Kubernetes Dashboard

```bash
minikube addons enable dashboard
kubectl apply -f k8s/dashboard/
kubectl -n kubernetes-dashboard create token admin-user
minikube dashboard
```

> ⚠️ `admin-user` is bound to `cluster-admin`. Use it only in a local lab, never on a shared or production cluster.

## Security notes

- `secrets.yaml` contains only a placeholder. In real environments, create Secrets with `kubectl create secret` or a secret manager (Sealed Secrets, External Secrets + AWS Secrets Manager), and never commit them.
- Kubernetes Secrets are only base64-encoded, not encrypted. Encryption at rest and RBAC are needed in production.

## CI

Every push and pull request runs **kubeconform** in strict mode against the Kubernetes API schemas, catching invalid fields and typos before anything reaches a cluster.

## Roadmap

- [ ] Replace NGINX with a custom app image built and pushed by CI
- [ ] Package with Helm or Kustomize (dev/prod overlays)
- [ ] Deploy to a managed cluster (EKS or AKS)
- [ ] Network Policies and Pod Security Standards

---

**Maikon Marquez** · Cloud & DevOps Engineer · [LinkedIn](https://www.linkedin.com/in/maikon-marquez) · [Medium](https://medium.com/@maikonjm)
