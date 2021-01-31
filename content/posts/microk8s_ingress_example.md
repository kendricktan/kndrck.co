---
title: Microk8s With Ingress Example
date: 2019-04-02
author: Kendrick Tan
categories: ["devops"]
---

## Prelude
This post is merely a reference guide for me to setup Kubernetes with Ingress.

### What's The Difference Between Ingress And A Loadbalancer?
LoadBalancer lives on L4 on the OSI model, Ingress services lives on L7 of the OSI model.

Use ingress for a single app with a single domain to be mapped to an IP address, use ingress to map multiple subdomains to multiple apps within your cluster.

For nicer infographs, refer to [Mathhew Palmer's blogpost](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html).

## Steps
### 1. Install MicroK8s
[Microk8s quickstart link](https://microk8s.io/#quick-start)

```bash
sudo snap install microk8s --classic
microk8s.start
```

### 2. Install Ingress Dependent Resources
```bash
microk8s.kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

### 3. Enable Ingress Addon
```bash
microk8s.enable ingress
```

### 4. Run a local Docker Registry
You can skip this step if you wanna host your image on a public/private repository.
```bash
docker run -p 5000:5000 registry
```

### 5. Clone Example Repository And Build Docker Image
```bash
git clone https://github.com/kendricktan/microk8s-ingress-example
cd microk8s-ingress-example

docker build . -t my-microk8s-app
docker tag my-microk8s-app localhost:5000/my-microk8s-app
docker push localhost:5000/my-microk8s-app
```

### 6. Run Applications And Ingress
```
microk8s.kubectl apply -f bar-deployment.yml
microk8s.kubectl apply -f foo-deployment.yml
microk8s.kubectl apply -f ingress.yml
```

#### 7. Expose Deployments to Ingress
If you skip this step you'll get a 503 service unavailable
```
microk8s.kubectl expose deployment foo-app --type=LoadBalancer --port=8080
microk8s.kubectl expose deployment bar-app --type=LoadBalancer --port=8080
```

#### 8. Testing Endpoint Out
```
curl -kL https://127.0.0.1/bar
curl -kL https://127.0.0.1/foo
```
