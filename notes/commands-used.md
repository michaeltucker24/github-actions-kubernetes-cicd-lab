# Commands Used

This file documents the main commands used while building the GitHub Actions Kubernetes CI/CD lab.

## Repository Setup

```bash
git clone git@github.com:michaeltucker24/github-actions-kubernetes-cicd-lab.git
cd github-actions-kubernetes-cicd-lab
```

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

## Local Docker Build

```bash
docker build -t cicd-demo-app:local .
```

## Local Docker Test

```bash
docker run -d -p 8080:80 --name cicd-demo-app cicd-demo-app:local
```

## Verify Local Container

```bash
docker ps
docker logs cicd-demo-app
```

## Stop and Remove Local Container

```bash
docker stop cicd-demo-app
docker rm cicd-demo-app
```

## Kubernetes Deployment

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

## Verify Kubernetes Resources

```bash
kubectl get deployments
kubectl get pods
kubectl get services
kubectl rollout status deployment/cicd-demo-app
```

## Git Commands

```bash
git status
git add .
git commit -m "Initial CI/CD project structure"
git push origin main
```

## Notes

The GitHub Actions workflow builds and pushes the Docker image, connects to Kubernetes using a GitHub Actions secret, applies the Kubernetes manifests, and verifies the rollout.