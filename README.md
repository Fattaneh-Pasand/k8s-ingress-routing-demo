## 📘 Kubernetes Ingress Routing Demo with Flux & Kind
# 🔹 Overview

This project simulates a production-like Kubernetes setup on a local machine using kind.

The goal is to demonstrate how to:

Run multiple microservices (foo, bar) behind a single entrypoint (Ingress).

Route all external traffic through the Ingress Controller.

Scale services and distribute traffic evenly across replicas.

Manage configuration with Flux (GitOps) for automation and reliability.

Map the setup to real-world cloud environments like AWS EKS, GKE, AKS.

# 🔹 Architecture
1. Microservices

Two independent Deployments:

foo → responds with "hello foo"

bar → responds with "hello bar"

Each Deployment runs 2 replicas for scalability.

2. Service Discovery & Load Balancing

Each Deployment is exposed internally via a ClusterIP Service.

Kubernetes Services handle load balancing across replicas.

3. Ingress Controller (Single Entrypoint)

An Ingress resource defines routing rules:

/foo → forwards traffic to foo service

/bar → forwards traffic to bar service

Requests from outside the cluster always pass through the Ingress Controller (ingress-nginx).

Flow of traffic:

Client → Ingress Controller → Service → Pod (replica)


This simulates how cloud Load Balancers work in managed Kubernetes environments.

4. GitOps with Flux

Flux continuously watches a GitHub repository for manifests.

When changes are pushed, Flux reconciles the cluster state with Git.

Flux controllers used:

Source Controller → fetches Git/Helm sources.

Kustomize Controller → applies kustomizations.

Helm Controller → manages Helm releases.

Notification Controller → sends events (Slack, GitHub, etc).

5. Scalability

Both foo and bar run multiple replicas.

Kubernetes distributes incoming requests across replicas automatically.

6. Security & Best Practices (Optional Extensions)

TLS termination with cert-manager (self-signed or ACM).

Secrets injection into pods (e.g., API_KEY).

Namespace separation for clean management:

flux-system → Flux controllers & sources.

ingress-nginx → ingress controller.

default or other namespaces → applications.

# 🔹 Setup & Steps
1. Create Kind Cluster
kind create cluster --name demo-ingress --config kind-cluster.yaml

2. Bootstrap Flux

Flux installs itself and commits manifests into GitHub.

flux bootstrap github \
  --owner=Fattaneh-Pasand \
  --repository=k8s-ingress-routing-demo \
  --branch=master \
  --path=clusters/kind-demo \
  --personal


--path → location in repo where cluster configuration lives.

To force Flux reconciliation after pushing changes:

flux reconcile source git flux-system

3. Deploy Ingress Controller

Define a HelmRepository in the flux-system namespace (for sources).

Deploy ingress-nginx in the ingress-nginx namespace.

Since kind does not provide a cloud LoadBalancer, expose the controller manually:

kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 8080:80


Now you can test routes:

http://localhost:8080/foo → "hello foo"

http://localhost:8080/bar → "hello bar"

All external traffic flows through the ingress-nginx controller, which then routes requests internally.

# 🔹 Cloud Mapping

Local (kind): ingress-nginx acts as the entrypoint; traffic is exposed via port-forward.

Cloud (AWS/GCP/Azure):

The same Ingress manifest is used.

The cloud provider’s Load Balancer Controller provisions:

AWS → Application Load Balancer (ALB) with Route53 + ACM certificates.

GCP → HTTP(S) Load Balancer.

Azure → Application Gateway.

This shows that GitOps + Ingress manifests are cloud-agnostic and portable.

# 🔹 Project Highlights

✅ Microservices (foo/bar) deployed with replicas
✅ Routing and load balancing through Ingress Controller
✅ GitOps workflow with Flux
✅ Secure, production-like namespace separation
✅ Same manifests run locally and in cloud (EKS, GKE, AKS)

👉 This project demonstrates how Ingress + Flux + kind can be used to simulate a real-world production Kubernetes setup, where all external requests are routed through the Ingress Controller — just like in AWS, GCP, or Azure.