# Scenario description

In this project, I simulate a production-like Kubernetes setup on my local machine using kind.
The goal is to demonstrate how to expose multiple microservices behind a single entrypoint (Ingress), scale them, and keep the configuration clean and secure — just like in a real AWS EKS or GKE environment.

.

# 🚀 What the project demonstrates

1-Microservices

Two independent services (foo, bar) running as Deployments with 2 replicas each.

Each one returns different responses (“hello foo”, “hello bar”) to simulate different APIs.

2-Service discovery & load balancing

Each Deployment is exposed internally with a Kubernetes Service.

Kubernetes load-balances requests across replicas.

3-Ingress routing

An Ingress resource routes external requests:

/foo → foo service

/bar → bar service

Implemented by the ingress-nginx controller, which acts like an API gateway.

4-Scalability

Each app runs multiple replicas.

Shows how Kubernetes distributes traffic evenly.

5-Security & best practices (optional extension)

(Optional) TLS with cert-manager/self-signed certificate.

(Optional) Secrets injected into pods (API_KEY demo).

6-Cloud mapping

Locally, ingress-nginx handles routing.

On AWS, the same Ingress YAML would be implemented by the AWS Load Balancer Controller, provisioning an Application Load Balancer with Route53 + ACM certificates.

# steps:

1-create kind cluster
kind create cluster --name demo-ingress --config apps/kind-cluster.yaml

2- bootsrtap flux 
When you run flux bootstrap, Flux installs itself and commits YAML manifests into your GitHub repo.
The --path flag tells Flux where inside your repo to put those cluster configuration files.
Those files describe Flux itself (controllers, sync config).

flux bootstrap github \
  --owner=Fattaneh-Pasand \
  --repository=k8s-ingress-routing-demo \
  --branch=master\
  --path=clusters/kind-demo \
  --personal

3- writng ingress-controller 
HelmRepository CRDs are always created in some namespace.
By convention, we put all our Flux “sources” (GitRepositories, HelmRepositories, Buckets) in flux-system, so Flux controllers know where to find them.flux-system namespace → holds Flux plumbing + source definitions (Git/Helm repos).targetNamespace (e.g. ingress-nginx, default, monitoring) → where actual apps run.It keeps GitOps infra config separate from workloads, which is considered a best practice.

flux reconcile source git flux-system



In a Kustomization (kustomize) file, any resources: entry that is a folder must contain its own kustomization.yaml.

kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80


Kustomization lives in flux-system because it’s a Flux control object, not your app