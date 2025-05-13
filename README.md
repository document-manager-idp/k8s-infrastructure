# Kubernetes Infrastructure for *Document Manager*

This repository contains the declarative Kubernetes manifests that deploy the **Document Manager** platform:

* **Auth Service** – Node.js API for authentication and Cognito integration  
* **Business‑Logic Service** – Python / Flask API that mediates between the UI, Auth and DB layers  
* **DB Service** – Python / Flask wrapper around OpenSearch for vector‑enabled storage and search  
* **OpenSearch + Dashboards** – Backend datastore (stateful set) with an optional UI  
* **Ingress** – Single public entry‑point that routes traffic to the above services  
* **Argo CD Application** – GitOps controller that keeps the cluster in sync with `main`  
* **Kustomize** – Composition layer that stitches the individual manifests together

---

## Quick start

### Prerequisites

* A running Kubernetes cluster (local or cloud)
* `kubectl` and `kustomize` (or `kubectl` ≥1.21 with `-k` support)
* An NGINX Ingress controller installed
* Two Kubernetes secrets already created in the **default** namespace:

```bash
kubectl create secret generic cognito-secret \
  --from-literal=COGNITO_SECRET=<your_value>

kubectl create secret generic opensearch-initial-admin-password \
  --from-literal=OPENSEARCH_INITIAL_ADMIN_PASSWORD=<your_password>
```

---

### GitOps with Argo CD

1. Install Argo CD in the cluster (see Argo CD docs).
2. Apply the application manifest:

```bash
kubectl apply -f argocd-document-manager.yaml
```

3. In the Argo CD UI, sync the **document-manager** application.  
   Argo CD will create / replace all resources and keep them in sync with the `main` branch.

Kustomize expands `kustomization.yaml`, which in turn pulls in every deployment, service, PVC and the ingress.

---

## Access points

| URL path | Backend service |
|----------|-----------------|
| `/auth` | **auth-service** on port 3000 |
| `/api` | **business‑logic-service** on port 5000 |
| `/dashboard` | **opensearch‑dashboards** on port 5601 |

These routes are configured in `ingress.yaml`.  

---

## Cleaning up

To remove everything that was installed by this repo:

```bash
argocd app delete document-manager     # or delete via UI
```
```bash
helm install monitoring \
   prometheus-community/kube-prometheus-stack \
   --namespace monitoring \
   --values monitoring/values.yaml
   
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```
