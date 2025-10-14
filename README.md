# JupyterHub on Minikube - CMCC Exercise

This repository contains a simple Kubernetes setup that runs **JupyterHub** with two persistent volumes:

- **`jupyterhub-data`** – mounted as read-only so all users can access shared input data.  
- **`jupyterhub-notebooks`** – mounted as read-write, with separate folders automatically created for each user’s notebooks and results.

You can create servers and work on it, it's functional.
Authentication uses the Github OAuth.  
Each user session launches directly into **JupyterLab**.

---

## Prerequisites

- A local Kubernetes cluster (tested on **Minikube**)  
- `kubectl` configured to connect to it  
- Recommended: **NGINX Ingress** add-on enabled on Minikube  

---

## Quick Start

First, clone the repo to your local machine.

1. **Start Minikube and enable Ingress**
   ```bash
   minikube start
   minikube addons enable ingress
   ```

Before deploying manifests, you need to configure your OAuth file by:
Go to github and create secret ID and secret password from Settings > Developer Settings > OAuth. Paste your info to the oauth-secret.yaml file.

2. **Deploy the manifests**
   ```bash
   kubectl apply -f k8s/storage.yaml
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   kubectl apply -f k8s/ingress-svc.yaml
   kubectl apply -f k8s/oauth-secret.yaml
   ```
   The first startup may take a minute while JupyterHub and JupyterLab images are downloaded.

3. **Map the hostname**
   ```bash
   minikube ip
   ```
   Then edit `/etc/hosts` and add:
   ```
   <MINIKUBE_IP> jupyterhub.local
   ```

4. **If Ingress doesn’t respond**, adjust the Ingress controller Service (You don't need to do for now, service is already configured.):
   ```bash
   kubectl get svc ingress-nginx-controller -n ingress-nginx -o yaml > ingress-svc.yaml
   ```
   Open the file and change:
   ```
   spec.type: LoadBalancer
   ```
   Then apply it:
   ```bash
   kubectl apply -f ingress-svc.yaml
   ```

5. **Open a new terminal** and start a tunnel:
   ```bash
   sudo minikube tunnel
   ```

6. **Check the External IP**
   ```bash
   kubectl get svc -n ingress-nginx
   ```
   Update `/etc/hosts` accordingly:
   ```bash
   echo "EXTERNAL_IP jupyterhub.local" | sudo tee -a /etc/hosts
   ```

7. Visit <http://jupyterhub.local>  
   Log in with Github. 

It might take a few seconds to load the website. 

---

## Upgrading

Update your YAML files as needed and reapply them:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/jupyterhub
```

If you change PVC sizes, delete and recreate them (Minikube HostPath volumes can’t be resized):
```bash
kubectl delete -f k8s/storage.yaml
kubectl apply -f k8s/storage.yaml
```

> **Note:** Deleting a PVC removes its data — back up notebooks first.

---

## Uninstall

To remove all resources:
```bash
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/storage.yaml
```

PersistentVolumes created by Minikube may need manual cleanup:
```bash
kubectl delete pv <NAME>
```

You may also want to stop and delete minikube.
```bash
minikube stop
minikube delete
```

---

## Customization

- **PVC sizes** → Open `k8s/storage.yaml`. Each `PersistentVolumeClaim` has a `spec.resources.requests.storage` field – edit the value (e.g. `5Gi`, `20Gi`) to change the requested size.
- **Mount paths** → In `k8s/deployment.yaml`, the hub container mounts the PVCs under `spec.template.spec.containers[].volumeMounts`.
- Change `mountPath` to the new path, then update the ConfigMap’s Python config (`data_path`, `notebooks_path`) to match.
- **Github OAuth parameteres** → Credentials live in the `jupyterhub-oauth` Secret (`k8s/oauth-secret.yaml`). Replace the `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` stringData values with your own, then `kubectl apply -f k8s/oauth-secret.yaml`.
- `OAUTH_CALLBACK_URL` (in the deployment env) must match the URL you registered with GitHub.
- `JUPYTERHUB_ALLOWED_LOGINS` lets you restrict access to specific GitHub usernames (comma-separated). Leave it empty to allow anyone with access to the OAuth app.
- `JUPYTERHUB_ADMIN_LOGINS` sets hub admins (also comma-separated usernames).
- **Spawner type** → currently uses `SimpleLocalProcessSpawner` (no system users required)  
- **Ingress host/class** → adjust the Ingress section in `k8s/service.yaml`  
- **Resource limits** → add CPU/Memory requests in `k8s/deployment.yaml`

---

## Optional Improvements
- **OAuth with Github** -> works, you need to edit your secrets in the ouath-secret.yaml file.
- **Readiness/Liveness probes** → check `/hub/health` for Hub status  
- **Security** → ***runs as root*** right now, you can switch to non-root with this configuration: (`runAsUser: 1000`, `fsGroup: 100`).  
  If you encounter permission issues, remove or adjust the `securityContext`.

---

## Troubleshooting

**Nothing loads at <http://jupyterhub.local>**  
- Check the `/etc/hosts` entry matches `minikube ip`  
- Verify Ingress is enabled and pods are running  

**Ingress unreachable (common with Docker driver)**  
Try one of these:
```bash
# Option A – temporary port forward
kubectl port-forward svc/jupyterhub 8000:8000
# Open http://localhost:8000/

# Option B – tunnel (macOS/Linux)
sudo minikube tunnel
```

Then check:
```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx
```
And update `/etc/hosts` so `jupyterhub.local` points to that IP.

---

## Working with Shared Data

Inside each notebook session:

- Shared, read-only data → `/srv/jupyterhub/input-data`  
- User-specific workspace → `/srv/jupyterhub/notebooks/<username>`

You can preload datasets by adding files to the `jupyterhub-data` PVC (for example via a helper pod).

