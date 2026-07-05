# Troubleshooting Notes

This file documents common issues encountered or expected while building the GitHub Actions Kubernetes CI/CD pipeline lab.

## Issue 1: GitHub Actions Workflow Fails Because Secrets Are Missing

**Problem:**  
The workflow fails during Docker Hub login or Kubernetes configuration.

**Likely Cause:**  
Required GitHub Actions secrets have not been created yet.

**Required Secrets:**

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
KUBE_CONFIG
```

**Fix:**  
Add the required secrets in the GitHub repository under:

```text
Settings > Secrets and variables > Actions > New repository secret
```

## Issue 2: Docker Image Does Not Pull in Kubernetes

**Problem:**  
The Kubernetes pod fails with an image pull error.

**Likely Cause:**  
The image name in `k8s/deployment.yaml` does not match the image pushed by GitHub Actions.

**Fix:**  
Confirm the image name is consistent in both places:

```yaml
image: michaeltucker24/cicd-demo-app:latest
```

and:

```yaml
tags: ${{ secrets.DOCKERHUB_USERNAME }}/cicd-demo-app:latest
```

## Issue 3: Kubernetes Rollout Does Not Complete

**Problem:**  
The workflow hangs or fails during:

```bash
kubectl rollout status deployment/cicd-demo-app
```

**Likely Cause:**  
The pod is not becoming ready, the image cannot be pulled, or the deployment has a manifest issue.

**Fix:**

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## Issue 4: Application Is Not Reachable in Browser

**Problem:**  
The pod is running, but the app cannot be reached from the browser.

**Likely Cause:**  
The AWS security group may not allow inbound traffic on the NodePort.

**Fix:**  
Allow inbound TCP traffic on:

```text
30080
```

Then test:

```text
http://<EC2-PUBLIC-IP>:30080
```

## Issue 5: Kubeconfig Secret Fails to Decode

**Problem:**  
The workflow fails during Kubernetes access configuration.

**Likely Cause:**  
The `KUBE_CONFIG` secret was not base64 encoded correctly.

**Fix:**  
Regenerate the kubeconfig secret value from the EC2 server and add it again to GitHub Actions secrets.

## Lessons Learned

This project demonstrates the importance of validating each layer of a CI/CD deployment separately:

1. Confirm the app works locally.
2. Confirm the Docker image builds.
3. Confirm Kubernetes manifests work manually.
4. Confirm registry authentication works.
5. Confirm GitHub Actions can connect to Kubernetes.
6. Confirm the rollout succeeds after automation.