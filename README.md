# GitHub Actions Kubernetes CI/CD Lab

This project demonstrates a complete CI/CD pipeline that builds a Docker image, pushes it to Docker Hub, deploys the application to Kubernetes, and verifies the rollout using GitHub Actions.

The deployment target for this lab is a single-node K3s Kubernetes cluster running on an AWS EC2 Ubuntu server.

---

## Project Overview

The goal of this project is to simulate a practical Junior DevOps CI/CD workflow using common tools used in real environments.

The pipeline performs the following actions:

```text
Developer pushes code to GitHub
        |
        v
GitHub Actions workflow starts
        |
        v
Docker image is built
        |
        v
Image is pushed to Docker Hub
        |
        v
GitHub Actions connects to Kubernetes
        |
        v
Kubernetes manifests are applied
        |
        v
Deployment rollout is verified
```

---

## Architecture

```text
Local Machine
     |
     | git push
     v
GitHub Repository
     |
     | triggers workflow
     v
GitHub Actions
     |
     | builds Docker image
     v
Docker Hub
     |
     | Kubernetes pulls image
     v
K3s Cluster on AWS EC2
     |
     | Deployment + Service
     v
Application running in Kubernetes
```

---

## Environment

| Component | Details |
|---|---|
| Cloud Provider | AWS |
| Compute | EC2 Ubuntu Server |
| Kubernetes | K3s single-node cluster |
| CI/CD | GitHub Actions |
| Container Registry | Docker Hub |
| Container Runtime | Docker |
| Application Type | Static web application served by Nginx |
| Deployment Method | `kubectl apply` |
| Service Type | NodePort |

---

## Tools and Technologies Used

| Tool | Purpose |
|---|---|
| Git | Version control |
| GitHub | Source code hosting |
| GitHub Actions | CI/CD automation |
| Docker | Container image build |
| Docker Hub | Container image registry |
| Kubernetes | Container orchestration |
| K3s | Lightweight Kubernetes distribution |
| kubectl | Kubernetes management |
| AWS EC2 | Kubernetes server host |
| Nginx | Static web server |

---

## Repository Structure

```text
github-actions-kubernetes-cicd-lab/
│
├── README.md
├── index.html
├── styles.css
├── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
└── notes/
    ├── commands-used.md
    └── troubleshooting.md
```

---

## Application

The application is a simple static web page served by Nginx.

The Docker image is built from this Dockerfile:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
COPY styles.css /usr/share/nginx/html/styles.css

EXPOSE 80
```

---

## Kubernetes Deployment

The application is deployed to Kubernetes using a Deployment and a NodePort Service.

The Deployment runs two replicas of the application:

```yaml
replicas: 2
```

The Service exposes the application through NodePort:

```yaml
type: NodePort
nodePort: 30080
```

During the lab, the application was accessible through:

```text
http://<EC2-PUBLIC-IP>:30080
```

---

## GitHub Actions Workflow

The GitHub Actions workflow is located at:

```text
.github/workflows/deploy.yml
```

The workflow performs the following steps:

```text
Checkout repository
        |
        v
Log in to Docker Hub
        |
        v
Set up Docker Buildx
        |
        v
Build and push Docker image
        |
        v
Set up kubectl
        |
        v
Configure Kubernetes access
        |
        v
Deploy to Kubernetes
        |
        v
Verify rollout
```

---

## GitHub Actions Secrets

The workflow requires the following repository secrets:

| Secret | Purpose |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub username used for image tagging and login |
| `DOCKERHUB_TOKEN` | Docker Hub access token used by GitHub Actions |
| `KUBE_CONFIG` | Base64-encoded kubeconfig used to connect to the K3s cluster |

The `KUBE_CONFIG` secret allows GitHub Actions to connect to the Kubernetes API server and run `kubectl` commands.

---

## Docker Image Architecture Fix

Because the local machine used for development was a Mac, the Docker image initially built for the wrong CPU architecture.

The Kubernetes pods failed with:

```text
exec format error
```

The fix was to build and push the image for the EC2 server architecture:

```bash
docker buildx build --platform linux/amd64 -t michaeltucker24/cicd-demo-app:latest --push .
```

The GitHub Actions workflow was also updated to build for `linux/amd64`:

```yaml
platforms: linux/amd64
```

This ensured the image could run correctly on the AWS EC2 Kubernetes node.

---

## Kubeconfig Public IP Fix

The default K3s kubeconfig pointed to a local or private address such as:

```text
127.0.0.1
```

or:

```text
172.31.x.x
```

That worked from inside the EC2 server but did not work from GitHub Actions.

The kubeconfig used by GitHub Actions needed to point to the EC2 public IP:

```text
https://<EC2-PUBLIC-IP>:6443
```

A corrected kubeconfig value was generated and stored as the `KUBE_CONFIG` GitHub Actions secret.

---

## K3s TLS Certificate Fix

After updating the kubeconfig to use the EC2 public IP, the workflow reached the Kubernetes API server but failed with a TLS certificate error.

The certificate was valid for internal addresses but not the public IP.

To fix this, the EC2 public IP was added as a K3s TLS Subject Alternative Name.

A K3s config file was created:

```yaml
tls-san:
  - <EC2-PUBLIC-IP>
```

Then K3s was restarted:

```bash
sudo systemctl restart k3s
```

After this change, GitHub Actions was able to connect securely to the Kubernetes API server.

---

## Manual Deployment Validation

Before relying on automation, the Kubernetes deployment was tested manually.

The manifests were copied to the EC2 server and applied with:

```bash
sudo kubectl apply -f k8s/deployment.yaml
sudo kubectl apply -f k8s/service.yaml
```

The deployment was verified with:

```bash
sudo kubectl get deployments
sudo kubectl get pods
sudo kubectl get services
sudo kubectl rollout status deployment/cicd-demo-app
```

This confirmed the Kubernetes manifests worked before enabling the automated pipeline.

---

## CI/CD Validation

After the GitHub Actions secrets were configured and the K3s certificate issue was resolved, the workflow successfully completed.

The successful workflow confirmed that GitHub Actions could:

1. Build the Docker image.
2. Push the image to Docker Hub.
3. Connect to the K3s Kubernetes cluster.
4. Apply the Kubernetes manifests.
5. Verify the deployment rollout.

---

## Security Note

For this lab, the Kubernetes API port was temporarily opened so GitHub Actions could reach the K3s API server:

```text
TCP 6443
```

This was done only for lab purposes.

After documentation is complete, the EC2 instance and related security group should be deleted or locked down to avoid exposing the Kubernetes API publicly.

---

## Key Troubleshooting Scenarios

| Issue | Cause | Fix |
|---|---|---|
| Pods showed `ImagePullBackOff` | Docker image was not pushed to Docker Hub yet | Built and pushed the image to Docker Hub |
| Pods failed with `exec format error` | Image was built for the wrong architecture | Rebuilt image using `linux/amd64` |
| GitHub Actions connected to `127.0.0.1` | Kubeconfig used local server address | Replaced kubeconfig server address with EC2 public IP |
| `base64: invalid input` | KUBE_CONFIG secret was copied incorrectly | Regenerated and copied kubeconfig directly to clipboard |
| TLS certificate error | K3s certificate did not include public IP | Added EC2 public IP as K3s TLS SAN |
| `kubectl` permission denied on EC2 | K3s kubeconfig requires elevated access | Used `sudo kubectl` on the EC2 server |

---

## Skills Demonstrated

This project demonstrates practical Junior DevOps skills including:

- Building Docker images
- Tagging and pushing images to Docker Hub
- Writing Kubernetes Deployment and Service manifests
- Running a lightweight Kubernetes cluster with K3s
- Deploying workloads to Kubernetes
- Creating GitHub Actions CI/CD workflows
- Managing GitHub Actions repository secrets
- Using kubeconfig for remote Kubernetes access
- Troubleshooting Kubernetes pod failures
- Troubleshooting CI/CD deployment errors
- Fixing Docker architecture mismatches
- Resolving Kubernetes API certificate issues

---

## Project Result

The final result is a working CI/CD pipeline where a push to the GitHub repository triggers GitHub Actions to build, push, deploy, and verify the application in Kubernetes.

This project represents a realistic Junior DevOps workflow using GitHub Actions, Docker, Docker Hub, Kubernetes, K3s, and AWS EC2.