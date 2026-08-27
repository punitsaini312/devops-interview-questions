01-kubernetes-ingress.md I'll teach you **Gateway API** now, formatted cleanly so you can copy-paste everything into a `.md` file for your GitHub notes. No fluff — just what you need.

---

# Kubernetes Gateway API — Complete Notes

## Table of Contents
1. [What is Gateway API?](#1-what-is-gateway-api)
2. [Why Did Kubernetes Create Gateway API?](#2-why-did-kubernetes-create-gateway-api)
3. [Ingress vs Gateway API — Side by Side](#3-ingress-vs-gateway-api--side-by-side)
4. [The 3 Core Resources](#4-the-3-core-resources)
5. [How It Works — The Flow](#5-how-it-works--the-flow)
6. [Real Example — 10 Microservices](#6-real-example--10-microservices)
7. [Gateway API in EKS (AWS)](#7-gateway-api-in-eks-aws)
8. [Gateway API in GKE (Google Cloud)](#8-gateway-api-in-gke-google-cloud)
9. [Gateway API in Helm Charts](#9-gateway-api-in-helm-charts)
10. [TLS, Redirects, Rate Limiting](#10-tls-redirects-rate-limiting)
11. [Interview Cheat Sheet](#11-interview-cheat-sheet)
12. [Quick Commands](#12-quick-commands)

---

## 1. What is Gateway API?

**Gateway API** is the **modern replacement for Ingress**. It was created because Ingress had limitations that became painful at scale.

Think of it like this:
- **Ingress** = One receptionist with one rulebook (simple, but limited)
- **Gateway API** = A **hotel with separate roles** — one person owns the building, another manages the lobby, and receptionists handle specific guests

> **Gateway API is a set of Kubernetes CRDs (Custom Resources) that provide more powerful, flexible, and role-based traffic management than Ingress.**

---

## 2. Why Did Kubernetes Create Gateway API?

| Ingress Problem | Gateway API Solution |
|----------------|---------------------|
| One Ingress resource mixes routing rules + TLS + LB config | **Separation of concerns** — different roles manage different parts |
| No standard way to do advanced routing (header-based, weight-based) | Built-in support for **header routing, traffic splitting, retries** |
| Annotations are vendor-specific and messy | **First-class fields** in the API — no magic annotations |
| Hard to share one LB across multiple teams | **Route binding** — multiple teams can attach routes to one gateway |
| Limited to HTTP/HTTPS | Supports **TCP, UDP, TLS passthrough, gRPC** natively |

---

## 3. Ingress vs Gateway API — Side by Side

### Ingress Way (Old)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt"
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
            name: auth-service
            port:
              number: 80
```

**Problems:**
- TLS, redirects, rewrite — all hidden in **annotations**
- One person needs access to everything
- Hard to manage 50+ services

---

### Gateway API Way (New)
```yaml
# Step 1: Gateway — defines the entry point (like a Load Balancer)
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: production-gateway
  namespace: ingress-gateways
spec:
  gatewayClassName: nginx          # Which controller?
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "myapp.example.com"
    tls:
      mode: Terminate
      certificateRefs:
      - name: myapp-tls-secret
  - name: http
    protocol: HTTP
    port: 80
    hostname: "myapp.example.com"

---
# Step 2: HTTPRoute — defines routing rules (owned by app team)
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: auth-route
  namespace: auth-team            # App team owns this!
spec:
  parentRefs:
  - name: production-gateway
    namespace: ingress-gateways   # Attach to the shared gateway
  hostnames:
  - myapp.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /auth
    backendRefs:
    - name: auth-service
      port: 80

---
# Step 3: Another HTTPRoute for web team
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: web-team
spec:
  parentRefs:
  - name: production-gateway
    namespace: ingress-gateways
  hostnames:
  - myapp.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```

**Why this is better:**
- **Gateway** = Infrastructure team manages the LB, TLS, domain
- **HTTPRoute** = App teams manage their own routing in their own namespaces
- **No annotations magic** — everything is explicit YAML fields

---

## 4. The 3 Core Resources

| Resource | Who Manages It | What It Does | Analogy |
|---------|---------------|-------------|---------|
| **GatewayClass** | Cluster admin | Defines WHICH controller to use (nginx, traefik, etc.) | "We use NGINX as our controller" |
| **Gateway** | Platform/Infrastructure team | Defines the actual entry point — IP, port, TLS, hostname | "The hotel building with address and security" |
| **HTTPRoute** | App teams | Defines routing rules for THEIR service | "Each restaurant's reservation policy" |

```
┌─────────────────────────────────────────┐
│         GatewayClass                    │
│    "We use NGINX controller"            │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│         Gateway                         │
│  "Listen on 443 for myapp.example.com"  │
│  "Terminate TLS with this certificate"  │
└─────────────────┬───────────────────────┘
                  ▼
        ┌─────────┴─────────┐
        ▼                   ▼
   HTTPRoute (auth)     HTTPRoute (web)
   "/auth → auth-svc"   "/ → web-svc"
        ▼                   ▼
   [auth Pods]         [web Pods]
```

---

## 5. How It Works — The Flow

```
User Browser
     ↓
https://myapp.example.com/auth
     ↓
[Cloud Load Balancer]  ← Created by Gateway controller
     ↓
[Gateway Controller Pod]  ← Reads Gateway + HTTPRoute resources
     ↓
"HTTPRoute says /auth → auth-service"
     ↓
[auth-service ClusterIP]
     ↓
[auth Pod]
```

**Key difference from Ingress:**
- Ingress = One YAML with everything mixed together
- Gateway API = **Multiple YAMPs, separate concerns**, multiple teams can work independently

---

## 6. Real Example — 10 Microservices

### Gateway (Managed by Platform Team)
```yaml
# platform-team/gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: app-gateway
  namespace: ingress-gateways
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "*.example.com"
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-tls-secret
  - name: http
    protocol: HTTP
    port: 80
    hostname: "*.example.com"
```

### HTTPRoutes (Each Team Manages Their Own)

```yaml
# auth-team/auth-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: auth-route
  namespace: auth-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /auth
    backendRefs:
    - name: auth-service
      port: 80
```

```yaml
# web-team/web-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: web-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
```

```yaml
# org-team/org-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: org-route
  namespace: org-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /org
    backendRefs:
    - name: org-service
      port: 80
```

```yaml
# datasource-team/datasource-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: datasource-route
  namespace: datasource-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /datasource
    backendRefs:
    - name: datasource-service
      port: 80
```

```yaml
# kpi-team/kpi-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: kpi-route
  namespace: kpi-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /kpi
    backendRefs:
    - name: kpi-service
      port: 80
```

```yaml
# projectdocs-team/projectdocs-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: projectdocs-route
  namespace: projectdocs-team
spec:
  parentRefs:
  - name: app-gateway
    namespace: ingress-gateways
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /projectdocs
    backendRefs:
    - name: projectdocs-service
      port: 80
```

---

## 7. Gateway API in EKS (AWS)

AWS supports Gateway API through the **AWS Gateway API Controller**.

### Install the Controller
```bash
helm repo add eks https://aws.github.io/eks-charts
helm install gateway-api-controller eks/aws-gateway-api-controller \
  --namespace aws-gw-controller \
  --create-namespace
```

### GatewayClass for AWS
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: aws-alb
spec:
  controllerName: gateway.networking.k8s.io/aws-alb
```

### Gateway with AWS ALB
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: aws-gateway
  namespace: ingress-gateways
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  gatewayClassName: aws-alb
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: myapp.example.com
    tls:
      mode: Terminate
      certificateRefs:
      - name: myapp-tls
```

> **Note:** As of 2026, AWS Gateway API support is still evolving. Many EKS teams still use **Ingress with AWS Load Balancer Controller** or **AWS Gateway Controller** for full Gateway API support. Check the latest AWS documentation for current status.

---

## 8. Gateway API in GKE (Google Cloud)

GKE has **native Gateway API support** — it's built-in and very mature.

### GatewayClass in GKE
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: gke-l7-global-external-managed
spec:
  controllerName: networking.gke.io/gateway-class
```

### Gateway with GKE
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gke-gateway
  namespace: ingress-gateways
spec:
  gatewayClassName: gke-l7-global-external-managed
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: myapp.example.com
    tls:
      mode: Terminate
      certificateRefs:
      - name: myapp-ssl-cert
```

> **GKE Advantage:** Google manages the controller for you. Just apply the YAML and GKE creates the load balancer automatically.

---

## 9. Gateway API in Helm Charts

### Central Gateway Chart (Platform Team)
```yaml
# platform-chart/templates/gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: {{ .Values.gateway.name }}
  namespace: {{ .Values.gateway.namespace }}
spec:
  gatewayClassName: {{ .Values.gateway.className }}
  listeners:
  {{- range .Values.gateway.listeners }}
  - name: {{ .name }}
    protocol: {{ .protocol }}
    port: {{ .port }}
    hostname: {{ .hostname }}
    {{- if .tls }}
    tls:
      mode: {{ .tls.mode }}
      certificateRefs:
      - name: {{ .tls.secretName }}
    {{- end }}
  {{- end }}
```

```yaml
# platform-chart/values.yaml
gateway:
  name: production-gateway
  namespace: ingress-gateways
  className: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "*.example.com"
    tls:
      mode: Terminate
      secretName: wildcard-tls
  - name: http
    protocol: HTTP
    port: 80
    hostname: "*.example.com"
```

### Per-Microservice HTTPRoute Chart
```yaml
# microservice-chart/templates/httproute.yaml
{{- if .Values.gateway.enabled }}
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: {{ include "microservice.fullname" . }}-route
  namespace: {{ .Release.Namespace }}
spec:
  parentRefs:
  - name: {{ .Values.gateway.parentName }}
    namespace: {{ .Values.gateway.parentNamespace }}
  hostnames:
  - {{ .Values.gateway.hostname }}
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: {{ .Values.gateway.path }}
    backendRefs:
    - name: {{ include "microservice.fullname" . }}
      port: {{ .Values.service.port }}
{{- end }}
```

```yaml
# microservice-chart/values.yaml
gateway:
  enabled: true
  parentName: production-gateway
  parentNamespace: ingress-gateways
  hostname: app.example.com
  path: /auth    # Change per service: /web, /org, /kpi, etc.

service:
  port: 80
```

---

## 10. TLS, Redirects, Rate Limiting

### HTTP to HTTPS Redirect
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: redirect-route
spec:
  parentRefs:
  - name: app-gateway
  rules:
  - filters:
    - type: RequestRedirect
      requestRedirect:
        scheme: https
        statusCode: 301
```

### Header-Based Routing
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-version-route
spec:
  parentRefs:
  - name: app-gateway
  rules:
  - matches:
    - headers:
      - name: x-api-version
        value: v2
    backendRefs:
    - name: api-v2-service
      port: 80
  - matches:
    - headers:
      - name: x-api-version
        value: v1
    backendRefs:
    - name: api-v1-service
      port: 80
```

### Traffic Splitting (Canary Deployment)
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: canary-route
spec:
  parentRefs:
  - name: app-gateway
  rules:
  - backendRefs:
    - name: web-service-stable
      port: 80
      weight: 90
    - name: web-service-canary
      port: 80
      weight: 10
```

---

## 11. Interview Cheat Sheet

| Question | Answer |
|---------|--------|
| What is Gateway API? | The modern replacement for Ingress — a set of CRDs for advanced traffic management with separation of concerns |
| Why not just use Ingress? | Ingress mixes infrastructure and app routing in one YAML. Gateway API separates them so multiple teams can work independently |
| What are the 3 core resources? | **GatewayClass** (which controller), **Gateway** (the entry point/LB), **HTTPRoute** (routing rules) |
| Who manages the Gateway? | Platform/Infrastructure team |
| Who manages the HTTPRoute? | Application teams in their own namespaces |
| Can multiple teams share one Gateway? | **Yes** — each team attaches their HTTPRoute to the same Gateway using `parentRefs` |
| What protocols does it support? | HTTP, HTTPS, TCP, UDP, TLS passthrough, gRPC |
| Does it support traffic splitting? | **Yes** — built-in with `weight` field in `backendRefs` |
| Does it support header-based routing? | **Yes** — built-in with `matches.headers` |
| What is `parentRefs`? | The field in HTTPRoute that attaches it to a Gateway |
| What is `backendRefs`? | The field that defines which Service to route traffic to |
| What is `GatewayClass`? | Defines which controller implementation to use (nginx, traefik, gke, aws-alb) |
| Is Gateway API built into Kubernetes? | **Yes** — it's an official Kubernetes API (graduated to v1 in recent versions) |
| Do I need a separate controller? | **Yes** — just like Ingress needs an Ingress Controller, Gateway API needs a Gateway Controller |

---

## 12. Quick Commands

```bash
# Install Gateway API CRDs (if not already present)
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml

# Check installed GatewayClasses
kubectl get gatewayclass

# List all Gateways
kubectl get gateway --all-namespaces

# List all HTTPRoutes
kubectl get httproute --all-namespaces

# Describe a Gateway to see its status
kubectl describe gateway app-gateway -n ingress-gateways

# Check if HTTPRoute is attached to Gateway
kubectl describe httproute auth-route -n auth-team

# Get the external IP of the Gateway
kubectl get gateway app-gateway -n ingress-gateways -o jsonpath='{.status.addresses[0].value}'
```

---

## Summary Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    EXTERNAL USER                             │
│         https://myapp.example.com/auth                       │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              CLOUD LOAD BALANCER                             │
│         (AWS ALB / GCE LB / NGINX LB)                      │
│              Created by Gateway Controller                   │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              GATEWAY CONTROLLER (Pod)                        │
│         Reads Gateway + HTTPRoute resources                  │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   HTTPRoute           HTTPRoute          HTTPRoute
   /auth → auth      /web → web         /kpi → kpi
        ▼                 ▼                 ▼
   [auth-svc]        [web-svc]          [kpi-svc]
        ▼                 ▼                 ▼
   [auth Pods]       [web Pods]         [kpi Pods]
```

---

## What to Say in an Interview

> *"Gateway API is the modern evolution of Ingress, designed to solve the limitations of the original Ingress API. While Ingress mixes routing rules, TLS configuration, and load balancer settings in a single resource, Gateway API separates these concerns into three distinct resources: GatewayClass defines which controller to use, Gateway is managed by the platform team and defines the entry point with TLS and listeners, and HTTPRoute is managed by individual application teams who attach their routes to the shared Gateway. This allows multiple teams to independently manage their routing in their own namespaces while sharing a single load balancer and domain. It also supports advanced features like header-based routing, traffic splitting for canary deployments, and multiple protocols natively without vendor-specific annotations."*

---

Ready for **Istio** next, or want to practice/review anything here?
