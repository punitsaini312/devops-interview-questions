# What is Ingress? (The Big Picture)

Think of your Kubernetes cluster as a gated community with 10 houses (your microservices). 

- ClusterIP = Only people inside the community can visit the house (internal only)
- NodePort = You opened one gate to one house, but you need a separate gate for every house (messy, port range 30000-32767)
- LoadBalancer = You hired a security guard for EACH house (expensive, one LB per service)

Ingress = ONE main gate + ONE smart receptionist who looks at the visitor's request and routes them to the right house.

> Ingress is a set of rules that tells Kubernetes: "If someone asks for `/auth`, send them to the auth service. If they ask for `/web`, send them to the web service."

---

# Why Do We Need Ingress?

| Problem | How Ingress Solves It |
|---|---|
| 10 microservices = 10 LoadBalancers = | 1 Ingress = 1 LoadBalancer = |
| Every service has a different IP/port | One domain, clean URLs (`/auth`, `/web`) |
| No SSL/TLS termination | Ingress handles HTTPS |
| No path-based routing | `/api` → API service, `/` → Web service |
| No rate limiting, rewriting | Ingress can do these |

Real-world analogy: You don't build 10 separate front doors to your office building. You build ONE lobby with a directory board.

---

# The Two Things You MUST Know

## 1. Ingress Resource (The "Rule Book")

This is a Kubernetes YAML object. It defines the rules — like a config file. It does NOTHING by itself.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx        # ← Which controller should read this?
  rules:
  - host: myapp.example.com      # ← Domain name
    http:
      paths:
      - path: /auth              # ← If URL starts with /auth
        pathType: Prefix
        backend:
          service:
            name: auth-service   # ← Send to this K8s Service
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /org
        pathType: Prefix
        backend:
          service:
            name: org-service
            port:
              number: 80
      - path: /datasource
        pathType: Prefix
        backend:
          service:
            name: datasource-service
            port:
              number: 80
      - path: /kpi
        pathType: Prefix
        backend:
          service:
            name: kpi-service
            port:
              number: 80
      - path: /projectdocs
        pathType: Prefix
        backend:
          service:
            name: projectdocs-service
            port:
              number: 80
```

## 2. Ingress Controller (The "Doer")

The Ingress resource is just a piece of paper with rules. Someone needs to READ that paper and ACT on it.

That's the Ingress Controller — it's a Pod running inside your cluster that:
- Watches for Ingress resources
- Configures a real reverse proxy (usually NGINX)
- Actually routes the traffic

### Popular Ingress Controllers:

| Controller | Who Uses It |
|---|---|
| NGINX Ingress Controller | Most popular, open-source |
| AWS ALB Ingress Controller | EKS (Amazon) |
| GCE Ingress Controller | GKE (Google) |
| Traefik | Cloud-native, auto-SSL |
| HAProxy | High performance |

> Key Interview Point: Without an Ingress Controller, your Ingress resource is useless — like having a traffic rulebook with no traffic police.

---

# How It Works (The Flow)

```text
User Browser
     ↓
myapp.example.com/auth
     ↓
[Cloud Load Balancer]  ← In EKS/GKE, this is created automatically
     ↓
[Ingress Controller Pod]  ← Reads your Ingress rules
     ↓
"Oh, /auth? → auth-service"
     ↓
[auth-service ClusterIP]  ← K8s Service
     ↓
[auth Pod]  ← Your actual app
```

---

# Ingress in EKS (AWS)

AWS has its own flavor: AWS Load Balancer Controller (formerly ALB Ingress Controller).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing   # Public-facing LB
    alb.ingress.kubernetes.io/target-type: ip           # Target pods directly
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
spec:
  ingressClassName: alb   # ← Tells AWS controller to handle this
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: auth-service
            port:
              number: 80
```

What happens in EKS:
1. You install the AWS Load Balancer Controller (Helm chart or YAML)
2. You create the Ingress with `ingressClassName: alb`
3. AWS automatically creates an Application Load Balancer (ALB) in your AWS account
4. The ALB routes traffic to your pods

Install AWS LB Controller:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=my-cluster \
  --set serviceAccount.create=true
```

---

# Ingress in GKE (Google Cloud)

GKE is even simpler — it has built-in GCE Ingress Controller.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.global-static-ip-name: my-static-ip
    networking.gke.io/managed-certificates: my-ssl-cert
spec:
  ingressClassName: gce        # ← Or "gce-internal" for private
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: auth-service
            port:
              number: 80
```

What happens in GKE:
1. GKE has the controller pre-installed
2. You create the Ingress
3. Google Cloud automatically creates a Global HTTP(S) Load Balancer
4. You get Google-grade CDN, SSL, DDoS protection automatically

---

# Ingress in Helm Charts (Your 10 Microservices)

In real projects, you don't write one giant Ingress YAML. Each microservice has its own Helm chart, and the Ingress is often managed separately or included.

## Option 1: One Central Ingress (Recommended)

Create a separate Helm chart or a single Ingress file for all services:

```yaml
# templates/ingress.yaml (in a "platform" or "infra" Helm chart)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # Auto SSL
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - {{ .Values.domain }}
    secretName: tls-secret
  rules:
  - host: {{ .Values.domain }}
    http:
      paths:
      {{- range .Values.services }}
      - path: {{ .path }}
        pathType: Prefix
        backend:
          service:
            name: {{ .name }}
            port:
              number: {{ .port }}
      {{- end }}
```

values.yaml:

```yaml
domain: myapp.example.com
services:
  - name: web-service
    path: /
    port: 80
  - name: auth-service
    path: /auth
    port: 80
  - name: org-service
    path: /org
    port: 80
  - name: datasource-service
    path: /datasource
    port: 80
  - name: kpi-service
    path: /kpi
    port: 80
  - name: projectdocs-service
    path: /projectdocs
    port: 80
```

## Option 2: Each Microservice Has Its Own Ingress

Some teams prefer each service to define its own routing rules:

```yaml
# auth-service/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: auth-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: {{ include "auth-service.fullname" . }}
            port:
              number: 80
```

> Interview tip: Central Ingress is easier to manage. Per-service Ingress is more decoupled but can get messy with 10+ services.

---

# Common Ingress Annotations (Super Important)

Annotations are like settings for your Ingress Controller:

```yaml
metadata:
  annotations:
    # Rewrite /auth/login to /login before sending to backend
    nginx.ingress.kubernetes.io/rewrite-target: /

    # Enable SSL redirect (HTTP → HTTPS)
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"

    # CORS headers
    nginx.ingress.kubernetes.io/enable-cors: "true"

    # Timeout settings
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"

    # Use specific cert
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

---

# Quick Interview Cheat Sheet

| Question | Answer |
|---|---|
| What is Ingress? | A K8s API object that defines HTTP routing rules from outside to inside the cluster |
| Difference between Ingress and Ingress Controller? | Ingress = rules (YAML). Ingress Controller = the actual proxy that enforces those rules (a Pod) |
| Can Ingress work without a controller? | No. It's just a config object. |
| Why not just use LoadBalancer for everything? | One LoadBalancer per service = expensive. Ingress = one entry point for many services. |
| How does EKS handle Ingress? | AWS Load Balancer Controller creates an ALB automatically |
| How does GKE handle Ingress? | Built-in GCE Ingress Controller creates a Google Cloud Load Balancer |
| What is `ingressClassName`? | Tells K8s WHICH controller should handle this Ingress (nginx, alb, gce, etc.) |
| What is `pathType: Prefix`? | Matches if the URL starts with this path. `Exact` = exact match only. |

---

# The One Diagram to Remember

```text
┌─────────────────────────────────────────┐
│           EXTERNAL USER                 │
│     https://myapp.example.com/auth      │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│      CLOUD LOAD BALANCER                │
│   (AWS ALB / GCE LB / NGINX LB)         │
│         Created automatically           │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│      INGRESS CONTROLLER (Pod)           │
│   Reads Ingress rules, routes traffic   │
└─────────────────┬───────────────────────┘
                  ▼
        ┌─────────┴─────────┐
        ▼                   ▼
   /auth → auth-svc    /web → web-svc
        ▼                   ▼
   [auth Pods]         [web Pods]
```

---

# What to Say in an Interview

> "Ingress is a Kubernetes resource that acts as a smart entry point for external HTTP/HTTPS traffic. It allows path-based and host-based routing to multiple backend services using a single external endpoint. However, Ingress itself is just a set of rules — it requires an Ingress Controller, like NGINX or AWS ALB Controller, to actually process and route the traffic. In our microservices setup with 10 services like auth, web, org, etc., we use a single Ingress resource to route `/auth` to the auth service, `/web` to the web service, and so on, all behind one domain with SSL termination."

---

# Next up: Gateway API (the modern replacement for Ingress) and then Istio (service mesh). Want me to continue with Gateway API, or do you want to practice some of this first?
