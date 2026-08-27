 I'll teach you **Istio** the same way — no jargon, just what you need for interviews. Copy-paste ready for your `.md` notes.

---

# Istio Service Mesh — Complete Notes

## Table of Contents
1. [What is Istio?](#1-what-is-istio)
2. [Why Do We Need Istio?](#2-why-do-we-need-istio)
3. [Istio Architecture — The 4 Core Components](#3-istio-architecture--the-4-core-components)
4. [The Sidecar Pattern — How It Actually Works](#4-the-sidecar-pattern--how-it-actually-works)
5. [Istio CRDs — The Building Blocks](#5-istio-crds--the-building-blocks)
6. [Traffic Management — VirtualService & DestinationRule](#6-traffic-management--virtualservice--destinationrule)
7. [Security — mTLS, AuthorizationPolicy, PeerAuthentication](#7-security--mtls-authorizationpolicy-peerauthentication)
8. [Observability — Kiali, Grafana, Jaeger, Prometheus](#8-observability--kiali-grafana-jaeger-prometheus)
9. [Installing Istio on EKS / GKE](#9-installing-istio-on-eks--gke)
10. [Istio with Your 10 Microservices + Helm](#10-istio-with-your-10-microservices--helm)
11. [Real Interview Scenarios](#11-real-interview-scenarios)
12. [Interview Cheat Sheet](#12-interview-cheat-sheet)
13. [Quick Commands](#13-quick-commands)

---

## 1. What is Istio?

**Istio is a Service Mesh.** 

Think of your microservices as **employees in an office** who talk to each other all day:
- `web` calls `auth`
- `auth` calls `org`
- `kpi` calls `datasource`

**Without Istio:** They shout across the room directly. No one knows what they're saying. No security. No logs.

**With Istio:** Every employee gets a **personal assistant (sidecar)** who sits next to them. All communication goes through the assistant. The assistant can:
- **Log** every conversation
- **Encrypt** every conversation
- **Block** unauthorized calls
- **Retry** failed calls
- **Route** calls to different versions
- **Add timeouts** so no one waits forever

> **Istio is a layer of infrastructure that adds security, observability, and traffic management to microservices — without changing any application code.**

---

## 2. Why Do We Need Istio?

| Problem Without Istio | How Istio Solves It |
|----------------------|---------------------|
| Services talk over HTTP — anyone can intercept | **mTLS** — automatic encryption between all services |
| No idea which service is failing | **Observability** — metrics, logs, traces for every request |
| One bad service crashes everything | **Circuit breaker** — stop sending traffic to failing services |
| No way to do canary deployments | **Traffic splitting** — send 10% traffic to new version |
| Services have no timeouts | **Timeouts & retries** — automatic |
| No access control between services | **Authorization policies** — who can talk to whom |
| Need to add CORS headers in every app | **Envoy proxy** — add headers automatically |

> **Key Point:** Istio solves problems that **Ingress/Gateway API cannot**. Ingress handles **external → internal** traffic. Istio handles **internal → internal** traffic between microservices.

---

## 3. Istio Architecture — The 4 Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTROL PLANE                             │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   istiod    │  │  Istio CA   │  │  Pilot (xDS server) │ │
│  │             │  │  (citadel)  │  │                     │ │
│  │ - Reads     │  │ - Issues    │  │ - Pushes config     │ │
│  │   Istio CRDs│  │   mTLS      │  │   to all proxies    │ │
│  │ - Converts  │  │   certs     │  │ - Discovers         │ │
│  │   to Envoy  │  │             │  │   services          │ │
│  │   config    │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                              │
│  One pod: istiod (replaces older separate components)       │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Pushes config via xDS API
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     DATA PLANE                               │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│  │  App    │◄──►│ Sidecar │    │  App    │◄──►│ Sidecar │ │
│  │  (web)  │    │ (envoy) │    │  (auth) │    │ (envoy) │ │
│  └─────────┘    └────┬────┘    └─────────┘    └────┬────┘ │
│                      │                                │      │
│                      └────────────┬───────────────────┘      │
│                                   │                          │
│                              mTLS encrypted                  │
│                              All traffic                     │
└─────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | What It Does | Where It Runs |
|----------|-------------|---------------|
| **istiod** | The brain. Reads your Istio YAMLs, converts them to Envoy config, pushes to all sidecars, issues mTLS certificates | Control plane — one deployment in `istio-system` namespace |
| **Envoy Proxy (Sidecar)** | The personal assistant. Intercepts ALL network traffic to/from your app pod. Adds headers, encrypts, logs, routes | Data plane — one container inside EVERY app pod |
| **Ingress Gateway** | The entry door from outside world into the mesh. Replaces/replaces your Ingress Controller | `istio-system` namespace — separate from app sidecars |
| **Egress Gateway** | Controls traffic LEAVING the mesh to external services (databases, APIs) | `istio-system` namespace |

---

## 4. The Sidecar Pattern — How It Actually Works

### Before Istio (Direct Communication)

```
[web Pod] ──────HTTP──────► [auth Pod]
  10.0.1.5                   10.0.2.8
```

Problems:
- Plain HTTP — no encryption
- No logs of what was sent
- No retries if auth is down
- No way to split traffic

---

### After Istio (Sidecar Intercepts Everything)

```
┌─────────────────────────────────┐      ┌─────────────────────────────────┐
│           web Pod               │      │           auth Pod              │
│  ┌─────────┐  ┌─────────────┐  │      │  ┌─────────┐  ┌─────────────┐  │
│  │   App   │◄─┤  Envoy      │  │◄────►│  │  Envoy  │◄─┤    App      │  │
│  │ (web)   │  │  (sidecar)  │  │ mTLS  │  │(sidecar)│  │   (auth)    │  │
│  │ :8080   │  │  :15001     │  │       │  │ :15001  │  │   :8080     │  │
│  └─────────┘  └─────────────┘  │      │  └─────────┘  └─────────────┘  │
│         ▲            ▲         │      │        ▲            ▲           │
│         │            │         │      │        │            │           │
│    iptables      iptables      │      │   iptables      iptables        │
│    redirect      redirect      │      │   redirect      redirect        │
│         │            │         │      │        │            │           │
│    ALL traffic goes through     │      │   ALL traffic goes through      │
│         Envoy sidecar           │      │         Envoy sidecar           │
└─────────────────────────────────┘      └─────────────────────────────────┘
```

**How it works:**
1. Istio injects an **Envoy sidecar** container into every pod
2. **iptables rules** redirect ALL traffic through the sidecar
3. Your app thinks it's talking directly — but everything goes through Envoy
4. Envoy talks to `istiod` to get config (routes, certs, policies)
5. Envoy encrypts traffic with **mTLS** automatically

> **Your app code does NOT change. Zero lines of code modified.**

---

## 5. Istio CRDs — The Building Blocks

Istio adds these Custom Resources to Kubernetes:

| CRD | Purpose | Who Uses It |
|-----|---------|-------------|
| **Gateway** | Defines external entry point (like Ingress, but Istio-native) | Platform team |
| **VirtualService** | Routing rules — timeouts, retries, traffic split, headers | App team |
| **DestinationRule** | Policies for reaching a destination — load balancing, mTLS, subsets | App team |
| **ServiceEntry** | Register external services (RDS, S3, APIs) into the mesh | Platform team |
| **PeerAuthentication** | Enforce mTLS between services (strict/permissive) | Security team |
| **AuthorizationPolicy** | Who can call whom — access control | Security team |
| **RequestAuthentication** | Validate JWT tokens from external users | Security team |
| **Sidecar** | Customize sidecar scope (which namespaces it watches) | Platform team |

---

## 6. Traffic Management — VirtualService & DestinationRule

### VirtualService — "How to Route Traffic"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web-routing
  namespace: web-team
spec:
  hosts:
  - web-service          # Which service this applies to
  http:
  - match:
    - headers:
        x-canary:
          exact: "true"  # If header says canary=true
    route:
    - destination:
        host: web-service
        subset: v2       # Send to v2 subset
      weight: 100
  - route:
    - destination:
        host: web-service
        subset: v1       # Default: send to v1
      weight: 90
    - destination:
        host: web-service
        subset: v2       # 10% to v2 (canary)
      weight: 10
    timeout: 5s           # Wait max 5 seconds
    retries:
      attempts: 3         # Retry 3 times
      perTryTimeout: 2s   # Each try max 2 seconds
      retryOn: 5xx        # Retry on server errors
    corsPolicy:
      allowOrigins:
      - exact: "https://myapp.example.com"
      allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      allowHeaders:
      - authorization
      - content-type
      - x-request-id
      allowCredentials: true
```

### DestinationRule — "How to Reach the Destination"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web-destination
  namespace: web-team
spec:
  host: web-service       # Which service this applies to
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 10
    outlierDetection:     # Circuit breaker
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
    loadBalancer:
      simple: LEAST_CONN  # Load balancing strategy
    tls:
      mode: ISTIO_MUTUAL  # Enable mTLS automatically
  subsets:                # Define versions for routing
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

---

## 7. Security — mTLS, AuthorizationPolicy, PeerAuthentication

### Enable Strict mTLS (All Services Must Encrypt)

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT          # ALL mesh traffic MUST be encrypted
```

> **Permissive mode** = Allow both encrypted and plain traffic (for migration)
> **Strict mode** = Only encrypted traffic allowed

---

### Control Who Can Call Whom

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: auth-policy
  namespace: auth-team
spec:
  selector:
    matchLabels:
      app: auth-service    # Apply to auth-service pods
  action: ALLOW
  rules:
  - from:
    - source:
        principals:        # Only these services can call auth
        - "cluster.local/ns/web-team/sa/web-service"
        - "cluster.local/ns/org-team/sa/org-service"
  - to:
    - operation:
        methods:           # Only these HTTP methods allowed
        - GET
        - POST
        paths:
        - "/api/auth/*"
```

---

### Validate JWT from External Users

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  jwtRules:
  - issuer: "https://auth0.com"
    jwksUri: "https://myapp.auth0.com/.well-known/jwks.json"
```

---

## 8. Observability — Kiali, Grafana, Jaeger, Prometheus

Istio automatically collects data from every sidecar. You get 4 tools:

| Tool | What It Shows | URL (after install) |
|------|--------------|---------------------|
| **Prometheus** | Raw metrics (requests/sec, errors, latency) | `kubectl port-forward svc/prometheus 9090:9090 -n istio-system` |
| **Grafana** | Pretty dashboards from Prometheus data | `kubectl port-forward svc/grafana 3000:3000 -n istio-system` |
| **Jaeger** | Distributed tracing — follow one request across 10 services | `kubectl port-forward svc/jaeger 16686:16686 -n istio-system` |
| **Kiali** | Visual service map — see your mesh topology | `kubectl port-forward svc/kiali 20001:20001 -n istio-system` |

**Kiali shows you:**
- Which services talk to which
- Error rates on each connection
- Traffic flow animation
- mTLS status (padlock icon = encrypted)

---

## 9. Installing Istio on EKS / GKE

### Step 1: Download Istio
```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.21.0  # Version may vary
export PATH=$PWD/bin:$PATH
```

### Step 2: Install Istio with Demo Profile (includes observability tools)
```bash
istioctl install --set profile=demo -y
```

**Profiles:**
| Profile | Use Case | Includes |
|---------|----------|----------|
| `demo` | Learning / small clusters | Everything (Kiali, Grafana, Jaeger, Prometheus) |
| `default` | Production | Core only, no observability |
| `minimal` | CI/CD | Just istiod, no gateways |
| `empty` | Custom | Nothing, you add what you need |

### Step 3: Enable Automatic Sidecar Injection
```bash
# Label your namespace — any new pod gets a sidecar automatically
kubectl label namespace web-team istio-injection=enabled
kubectl label namespace auth-team istio-injection=enabled
kubectl label namespace org-team istio-injection=enabled
kubectl label namespace datasource-team istio-injection=enabled
kubectl label namespace kpi-team istio-injection=enabled
kubectl label namespace projectdocs-team istio-injection=enabled

# Restart existing pods to get sidecars
kubectl rollout restart deployment -n web-team
kubectl rollout restart deployment -n auth-team
# ... do for all namespaces
```

### Step 4: Verify Sidecars Are Injected
```bash
kubectl get pods -n web-team
# Output:
# NAME                        READY   STATUS
# web-service-abc-123         2/2     Running   ← 2/2 = app + sidecar
```

> **If you see 1/1** — sidecar injection failed. Check namespace label.

### Step 5: Install Ingress Gateway (for external traffic)
```bash
# Already included in demo profile
# Verify:
kubectl get svc istio-ingressgateway -n istio-system
# Get the external IP:
kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

---

## 10. Istio with Your 10 Microservices + Helm

### Structure

```
platform-chart/
├── templates/
│   ├── gateway.yaml          # Istio Gateway (entry point)
│   ├── peerauthentication.yaml  # Enforce mTLS
│   └── authorizationpolicy.yaml # Default deny-all
└── values.yaml

microservice-chart/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── virtualservice.yaml   # Routing rules
│   └── destinationrule.yaml  # Subsets, mTLS, circuit breaker
└── values.yaml
```

### Platform Chart — Gateway

```yaml
# platform-chart/templates/gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: app-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway    # Attach to Istio's ingress gateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: myapp-tls-secret
    hosts:
    - "myapp.example.com"
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "myapp.example.com"
```

### Platform Chart — Enforce mTLS

```yaml
# platform-chart/templates/peerauthentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

### Per Microservice — VirtualService

```yaml
# microservice-chart/templates/virtualservice.yaml
{{- if .Values.istio.enabled }}
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: {{ include "microservice.fullname" . }}
  namespace: {{ .Release.Namespace }}
spec:
  hosts:
  - {{ include "microservice.fullname" . }}
  http:
  - match:
    - uri:
        prefix: {{ .Values.istio.path }}
    route:
    - destination:
        host: {{ include "microservice.fullname" . }}
        port:
          number: {{ .Values.service.port }}
    timeout: {{ .Values.istio.timeout }}
    retries:
      attempts: {{ .Values.istio.retries.attempts }}
      perTryTimeout: {{ .Values.istio.retries.perTryTimeout }}
      retryOn: {{ .Values.istio.retries.retryOn }}
    corsPolicy:
      allowOrigins:
      - exact: "https://myapp.example.com"
      allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      - OPTIONS
      allowHeaders:
      - authorization
      - content-type
      - x-request-id
      - x-correlation-id
      allowCredentials: true
{{- end }}
```

### Per Microservice — DestinationRule

```yaml
# microservice-chart/templates/destinationrule.yaml
{{- if .Values.istio.enabled }}
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: {{ include "microservice.fullname" . }}
  namespace: {{ .Release.Namespace }}
spec:
  host: {{ include "microservice.fullname" . }}
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: {{ .Values.istio.connectionPool.maxConnections }}
      http:
        http1MaxPendingRequests: {{ .Values.istio.connectionPool.http1MaxPendingRequests }}
    outlierDetection:
      consecutiveErrors: {{ .Values.istio.outlierDetection.consecutiveErrors }}
      interval: {{ .Values.istio.outlierDetection.interval }}
      baseEjectionTime: {{ .Values.istio.outlierDetection.baseEjectionTime }}
    loadBalancer:
      simple: {{ .Values.istio.loadBalancer }}
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
{{- end }}
```

### values.yaml for Each Service

```yaml
# microservice-chart/values.yaml
service:
  port: 80

istio:
  enabled: true
  path: /auth        # Change per service: /web, /org, /kpi, etc.
  timeout: 10s
  retries:
    attempts: 3
    perTryTimeout: 3s
    retryOn: "gateway-error,connect-failure,refused-stream"
  connectionPool:
    maxConnections: 100
    http1MaxPendingRequests: 50
  outlierDetection:
    consecutiveErrors: 5
    interval: 30s
    baseEjectionTime: 30s
  loadBalancer: LEAST_CONN
```

---

## 11. Real Interview Scenarios

### Scenario 1: "Increase timeout for a slow API"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: datasource-route
spec:
  hosts:
  - datasource-service
  http:
  - route:
    - destination:
        host: datasource-service
    timeout: 30s        # Increase from default (no timeout)
```

### Scenario 2: "Add CORS headers"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web-route
spec:
  hosts:
  - web-service
  http:
  - route:
    - destination:
        host: web-service
    corsPolicy:
      allowOrigins:
      - exact: "https://myapp.example.com"
      allowMethods: ["GET", "POST", "PUT", "DELETE"]
      allowHeaders: ["authorization", "content-type", "x-request-id"]
      allowCredentials: true
```

### Scenario 3: "Canary deployment — 10% to v2"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: auth-canary
spec:
  hosts:
  - auth-service
  http:
  - route:
    - destination:
        host: auth-service
        subset: v1
      weight: 90
    - destination:
        host: auth-service
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: auth-subsets
spec:
  host: auth-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Scenario 4: "Circuit breaker — stop calling failing service"

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: kpi-circuitbreaker
spec:
  host: kpi-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5      # After 5 errors in a row...
      interval: 10s
      baseEjectionTime: 30s     # ...eject for 30 seconds
      maxEjectionPercent: 50    # Eject max 50% of pods
```

### Scenario 5: "Only web and auth can call org-service"

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: org-access
  namespace: org-team
spec:
  selector:
    matchLabels:
      app: org-service
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/web-team/sa/web-service"
        - "cluster.local/ns/auth-team/sa/auth-service"
```

---

## 12. Interview Cheat Sheet

| Question | Answer |
|---------|--------|
| What is Istio? | A service mesh that adds security, traffic management, and observability to microservices without code changes |
| What is a service mesh? | A dedicated infrastructure layer that handles service-to-service communication |
| What is the sidecar pattern? | A proxy container (Envoy) injected into every pod that intercepts all network traffic |
| What is istiod? | The Istio control plane — reads CRDs, converts to Envoy config, issues mTLS certs |
| What is Envoy? | The proxy that runs as a sidecar in every pod — does the actual routing, encryption, logging |
| Difference between Istio Gateway and Kubernetes Ingress? | Istio Gateway is Istio-native, works with VirtualService, supports advanced routing. Kubernetes Ingress is simpler and generic |
| What is mTLS? | Mutual TLS — both client and server verify each other's identity. Istio does this automatically between all mesh services |
| What is a VirtualService? | Defines routing rules — timeouts, retries, traffic split, headers, CORS |
| What is a DestinationRule? | Defines policies for reaching a service — load balancing, subsets, circuit breaker, mTLS settings |
| What is PeerAuthentication? | Controls mTLS enforcement — STRICT (must encrypt) or PERMISSIVE (allow both) |
| What is AuthorizationPolicy? | Access control — defines which services can call which other services |
| How do you enable sidecar injection? | Label namespace with `istio-injection=enabled`, then restart pods |
| What is traffic splitting? | Route percentage of traffic to different versions (canary) using weights in VirtualService |
| What is a circuit breaker? | Stop sending traffic to failing pods after consecutive errors |
| What observability tools come with Istio? | Prometheus (metrics), Grafana (dashboards), Jaeger (tracing), Kiali (topology map) |
| How does Istio handle external services? | ServiceEntry — registers external services (RDS, S3) into the mesh so they can be controlled |
| Can Istio replace Ingress? | **Partially** — Istio Gateway handles external entry, but many teams use both: Ingress for simple routing, Istio for mesh features |

---

## 13. Quick Commands

```bash
# Install Istio
istioctl install --set profile=demo -y

# Check Istio version
istioctl version

# Verify installation
istioctl verify-install

# Enable sidecar injection on namespace
kubectl label namespace <namespace> istio-injection=enabled

# Check if sidecar is injected (should see 2/2 ready)
kubectl get pods -n <namespace>

# Get ingress gateway external IP
kubectl get svc istio-ingressgateway -n istio-system

# Port-forward Kiali
kubectl port-forward svc/kiali 20001:20001 -n istio-system
# Open: http://localhost:20001

# Port-forward Grafana
kubectl port-forward svc/grafana 3000:3000 -n istio-system
# Open: http://localhost:3000

# Check proxy config for a pod
istioctl proxy-config cluster <pod-name> -n <namespace>

# Check proxy listeners
istioctl proxy-config listener <pod-name> -n <namespace>

# Check proxy routes
istioctl proxy-config route <pod-name> -n <namespace>

# Check mTLS status between services
istioctl authn tls-check <source-pod>.<namespace> <destination-service>.<namespace>

# Analyze Istio config for errors
istioctl analyze --all-namespaces

# Uninstall Istio
istioctl uninstall --purge
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
│              ISTIO INGRESS GATEWAY                           │
│         (Envoy proxy — entry to the mesh)                   │
│              TLS termination here                             │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    ISTIO CONTROL PLANE                       │
│                        istiod                                │
│         (reads CRDs, pushes config, issues certs)           │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      DATA PLANE                              │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│  │  App    │◄──►│  Envoy  │    │  App    │◄──►│  Envoy  │ │
│  │ (web)   │    │(sidecar)│    │ (auth)  │    │(sidecar)│ │
│  └─────────┘    └────┬────┘    └─────────┘    └────┬────┘ │
│                      │                                │      │
│                      └────────────┬───────────────────┘      │
│                                   │                          │
│                         mTLS encrypted                       │
│                    All traffic logged, routed, secured        │
└─────────────────────────────────────────────────────────────┘
```

---

## What to Say in an Interview

> *"Istio is a service mesh that we use to secure, manage, and observe traffic between our microservices. It works by injecting an Envoy sidecar proxy into every pod via automatic sidecar injection. All network traffic is intercepted by this proxy, which communicates with the Istio control plane called istiod. Istiod reads our Istio CRDs like VirtualService for routing rules, DestinationRule for load balancing and circuit breakers, and PeerAuthentication for mTLS enforcement. This gives us automatic encryption between services, traffic splitting for canary deployments, timeouts and retries, CORS handling, and fine-grained access control via AuthorizationPolicy — all without changing any application code. For observability, we use Kiali for service topology, Grafana for dashboards, Jaeger for distributed tracing, and Prometheus for metrics. We installed Istio using istioctl with the demo profile for learning, and in production we'd use the default profile. We enable sidecar injection by labeling namespaces with istio-injection=enabled and restart deployments."*

---

Ready to move on, or want to drill on any specific Istio topic?
