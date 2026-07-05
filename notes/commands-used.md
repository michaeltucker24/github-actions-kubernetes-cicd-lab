# Commands Used

This file documents the main commands used while building the GitHub Actions Kubernetes CI/CD lab.

---

## Repository Setup

```bash
git clone git@github.com:michaeltucker24/github-actions-kubernetes-cicd-lab.git
cd github-actions-kubernetes-cicd-lab
```

---

## Folder and File Creation

```bash
mkdir -p k8s .github/workflows notes

touch README.md
touch index.html
touch styles.css
touch Dockerfile
touch k8s/deployment.yaml
touch k8s/service.yaml
touch .github/workflows/deploy.yml
touch notes/commands-used.md
touch notes/troubleshooting.md
```

---

## Local Docker Build

```bash
docker build -t cicd-demo-app:local .
```

---

## Local Docker Test

```bash
docker run -d -p 8080:80 --name cicd-demo-app cicd-demo-app:local
```

---

## Verify Local Container

```bash
docker ps
docker logs cicd-demo-app
```

The application was tested locally in a browser at:

```text
http://localhost:8080
```

---

## Stop and Remove Local Test Container

```bash
docker stop cicd-demo-app
docker rm cicd-demo-app
```

---

## SSH Into EC2 Server

```bash
ssh -i ~/Downloads/k3s-cicd-demo-key.pem ubuntu@54.151.242.84
```

---

## Update EC2 Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Install K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

---

## Verify K3s Service

```bash
sudo systemctl status k3s
```

---

## Verify Kubernetes Node

```bash
sudo kubectl get nodes
```

---

## Verify Kubernetes System Pods

```bash
sudo kubectl get pods -A
```

---

## Copy Kubernetes Manifests to EC2

From the local machine:

```bash
scp -i ~/Downloads/k3s-cicd-demo-key.pem -r k8s ubuntu@54.151.242.84:~/
```

---

## Manual Kubernetes Deployment

From the EC2 server:

```bash
sudo kubectl apply -f ~/k8s/deployment.yaml
sudo kubectl apply -f ~/k8s/service.yaml
```

---

## Verify Kubernetes Resources

```bash
sudo kubectl get deployments
sudo kubectl get pods
sudo kubectl get services
sudo kubectl rollout status deployment/cicd-demo-app
```

---

## Docker Hub Login

```bash
docker logout
docker login
docker info | grep Username
```

---

## Build Docker Image for Local Testing

```bash
docker build -t michaeltucker24/cicd-demo-app:latest .
```

---

## Push Docker Image to Docker Hub

```bash
docker push michaeltucker24/cicd-demo-app:latest
```

---

## Build and Push Docker Image for EC2 Architecture

The first image failed in Kubernetes because it was built for the wrong CPU architecture.

The corrected build command was:

```bash
docker buildx build --platform linux/amd64 -t michaeltucker24/cicd-demo-app:latest --push .
```

---

## Restart Kubernetes Deployment

From the EC2 server:

```bash
sudo kubectl rollout restart deployment/cicd-demo-app
sudo kubectl rollout status deployment/cicd-demo-app
sudo kubectl get pods
```

---

## Check Failed Pod Details

```bash
POD=$(sudo kubectl get pods -l app=cicd-demo-app -o jsonpath='{.items[0].metadata.name}')
echo $POD
sudo kubectl describe pod $POD
sudo kubectl logs $POD
```

---

## Generate KUBE_CONFIG Secret

The K3s kubeconfig needed to be modified so GitHub Actions could connect to the EC2 public IP instead of `127.0.0.1` or the EC2 private IP.

From the local machine:

```bash
ssh -i ~/Downloads/k3s-cicd-demo-key.pem ubuntu@54.151.242.84 "sudo cat /etc/rancher/k3s/k3s.yaml | sed 's/127.0.0.1/54.151.242.84/g' | sed 's/172.31.43.216/54.151.242.84/g' | base64 -w 0" | pbcopy
```

The copied value was added to GitHub as the `KUBE_CONFIG` repository secret.

---

## Add Public IP to K3s TLS SAN

The K3s API certificate did not originally include the EC2 public IP.

The following configuration was added on the EC2 server:

```bash
sudo mkdir -p /etc/rancher/k3s

sudo tee /etc/rancher/k3s/config.yaml > /dev/null <<EOF
tls-san:
  - 54.151.242.84
EOF
```

Then K3s was restarted:

```bash
sudo systemctl restart k3s
```

Kubernetes was verified again:

```bash
sudo kubectl get nodes
```

---

## Re-run GitHub Actions Workflow

After updating the GitHub secrets and K3s TLS SAN configuration, the failed workflow was re-run from the GitHub Actions page.

The workflow completed successfully after the Kubernetes API connection and TLS issues were resolved.

---

## Git Commands

```bash
git status
git add .
git commit -m "Add initial CI/CD project files"
git push origin main
```

README update:

```bash
git status
git add README.md
git commit -m "Document CI/CD pipeline project"
git push origin main
```

Workflow architecture update:

```bash
git status
git add .github/workflows/deploy.yml
git commit -m "Build Docker image for amd64 in GitHub Actions"
git push origin main
```

---

## Final Application Test

The application was verified in a browser at:

```text
http://54.151.242.84:30080
```

---

## Notes

This project required validating each layer separately:

1. Local Docker build and container test.
2. Manual Kubernetes deployment to K3s.
3. Docker Hub image push.
4. GitHub Actions secret configuration.
5. Remote Kubernetes access through kubeconfig.
6. K3s TLS SAN update.
7. Successful automated CI/CD deployment.