 Ah, this makes **perfect sense** now! You were working in a **company where Istio was already set up** — you only touched `VirtualService` and `values.yaml`. You never saw the "plumbing" behind the walls. Let me show you **exactly** what your platform team set up, and how you'd replicate it for `kotabonsai.com` from scratch.

---

# Istio From Scratch — What Your Company Hid From You

## What You Were Actually Doing (The Full Picture)

Your company had this already set up (you never touched these):

```yaml
# ============================================
# FILE 1: Gateway (Platform team created this ONCE)
# Located in: istio-system namespace
# You NEVER saw this file
# ============================================
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: tecorelabs-gateway        # ← This is what your VirtualService referenced!
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway         # ← Attaches to the Istio Ingress Gateway pod
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: tecorelabs-tls-secret   # ← SSL cert for tecorelabs.com
    hosts:
    - "tecorelabs.com"            # ← This domain is allowed
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "tecorelabs.com"
```

```yaml
# ============================================
# FILE 2: Your VirtualService (This is what YOU edited)
# Located in: your service namespace (auth-team, org-team, etc.)
# ============================================
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: auth-route
  namespace: auth-team
spec:
  hosts:
  - "tecorelabs.com"             # ← Must match Gateway hosts
  gateways:
  - tecorelabs-gateway           # ← ← ← THIS IS THE MAGIC LINK!
  http:
  - match:
    - uri:
        prefix: /kap/auth        # ← ← ← This is what you added in values.yaml!
    route:
    - destination:
        host: auth-service
        port:
          number: 80
```

**Your `values.yaml` probably looked like this:**
```yaml
# values.yaml (your Helm chart)
domain: tecorelabs.com
project: kap
services:
  - name: auth
    path: /auth
  - name: org
    path: /org
```

And the Helm template generated:
- `hosts: ["tecorelabs.com"]`
- `gateways: ["tecorelabs-gateway"]`
- `match: uri: prefix: /kap/auth`

---

## The Missing Pieces (What Your Platform Team Did)

| File | What It Does | Where It Lives | Who Manages |
|------|-------------|----------------|-------------|
| **Gateway** | "I accept traffic for tecorelabs.com on port 443" | `istio-system` namespace | Platform team |
| **TLS Secret** | SSL certificate for `tecorelabs.com` | `istio-system` namespace | Platform team |
| **Istio Ingress Gateway Service** | AWS/GCP Load Balancer with public IP | `istio-system` namespace | Platform team |
| **DNS Record** | `tecorelabs.com` → LB IP | Route 53 / Cloud DNS | Platform team |
| **VirtualService** | "/kap/auth → auth-service" | Your namespace (`auth-team`) | **You** |

---

## Setting Up Istio for `kotabonsai.com` From Scratch

### Step 1: Install Istio (One-Time Setup)

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.21.0
export PATH=$PWD/bin:$PATH

# Install with demo profile (includes observability)
istioctl install --set profile=demo -y

# Verify
kubectl get pods -n istio-system
# You should see:
# istiod-xxx           1/1 Running
# istio-ingressgateway-xxx  1/1 Running
```

---

### Step 2: Get the Load Balancer URL (AWS Example)

```bash
# Get the external URL of Istio Ingress Gateway
kubectl get svc istio-ingressgateway -n istio-system

# Output:
# NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP
# istio-ingressgateway   LoadBalancer   10.100.200.1    a1b2c3d4.elb.amazonaws.com
```

**This `a1b2c3d4.elb.amazonaws.com` is your public entry point.**

---

### Step 3: Create SSL Certificate for `kotabonsai.com`

**Option A: Using cert-manager (Recommended)**
```yaml
# cert-manager ClusterIssuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@kotabonsai.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: istio
```

```yaml
# Certificate for kotabonsai.com
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: kotabonsai-tls
  namespace: istio-system
spec:
  secretName: kotabonsai-tls-secret    # ← This name goes in Gateway!
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - kotabonsai.com
  - www.kotabonsai.com
```

**Option B: Manual Certificate**
```bash
# If you have your own cert files
kubectl create secret tls kotabonsai-tls-secret \
  --cert=kotabonsai.crt \
  --key=kotabonsai.key \
  -n istio-system
```

---

### Step 4: Create the Gateway for `kotabonsai.com`

```yaml
# gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: kotabonsai-gateway          # ← You will reference this in VirtualService!
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway           # ← Attach to Istio's ingress gateway pod
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: kotabonsai-tls-secret   # ← From Step 3!
    hosts:
    - "kotabonsai.com"              # ← Your new domain!
    - "www.kotabonsai.com"
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "kotabonsai.com"
    - "www.kotabonsai.com"
```

Apply it:
```bash
kubectl apply -f gateway.yaml
```

---

### Step 5: Create DNS Record (Route 53 / Cloudflare / etc.)

| Record Type | Name | Value |
|------------|------|-------|
| A (or CNAME) | `kotabonsai.com` | `a1b2c3d4.elb.amazonaws.com` |
| CNAME | `www.kotabonsai.com` | `kotabonsai.com` |

> **Wait 5-10 minutes for DNS to propagate.**

---

### Step 6: Enable Sidecar Injection on Your Namespaces

```bash
# Create namespaces for your microservices
kubectl create namespace auth-team
kubectl create namespace org-team
kubectl create namespace web-team

# Enable automatic sidecar injection
kubectl label namespace auth-team istio-injection=enabled
kubectl label namespace org-team istio-injection=enabled
kubectl label namespace web-team istio-injection=enabled

# Verify
kubectl get namespace -L istio-injection
```

---

### Step 7: Deploy Your Microservices with VirtualService

Now this is the part **you already know!** But now you understand the full picture.

```yaml
# auth-virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: auth-route
  namespace: auth-team
spec:
  hosts:
  - "kotabonsai.com"                # ← Must match Gateway hosts!
  gateways:
  - kotabonsai-gateway              # ← ← ← LINK TO THE GATEWAY FROM STEP 4!
  http:
  - match:
    - uri:
        prefix: /kap/auth           # ← Your path: kotabonsai.com/kap/auth
    route:
    - destination:
        host: auth-service
        port:
          number: 80
```

```yaml
# org-virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: org-route
  namespace: org-team
spec:
  hosts:
  - "kotabonsai.com"
  gateways:
  - kotabonsai-gateway              # ← Same gateway, different namespace!
  http:
  - match:
    - uri:
        prefix: /kap/org            # ← kotabonsai.com/kap/org
    route:
    - destination:
        host: org-service
        port:
          number: 80
```

---

## The Complete Flow for `kotabonsai.com/kap/auth`

```
User types: https://kotabonsai.com/kap/auth
     ↓
DNS resolves kotabonsai.com → AWS ELB IP
     ↓
┌─────────────────────────────────────────┐
│    AWS Load Balancer (created by        │
│    istio-ingressgateway Service)        │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│    Istio Ingress Gateway Pod            │
│    (Envoy proxy in istio-system)        │
│    Checks: Host = kotabonsai.com ✓      │
│    Uses: kotabonsai-tls-secret ✓        │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│    "Which VirtualService matches        │
│     kotabonsai.com + /kap/auth?"       │
│                                         │
│    Found: auth-route VirtualService     │
│    in auth-team namespace               │
│    (linked via gateways: kotabonsai-    │
│     gateway)                            │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│    Envoy Sidecar in auth Pod            │
│    (injected automatically)             │
│    Routes to: auth-service:80           │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│    auth Pod (your actual application)   │
└─────────────────────────────────────────┘
```

---

## Your Company's Setup vs. New Setup

| Component | Your Company (`tecorelabs.com`) | Your New Setup (`kotabonsai.com`) |
|----------|--------------------------------|----------------------------------|
| **Gateway name** | `tecorelabs-gateway` | `kotabonsai-gateway` |
| **Gateway namespace** | `istio-system` | `istio-system` |
| **TLS secret** | `tecorelabs-tls-secret` | `kotabonsai-tls-secret` |
| **Domain in Gateway hosts** | `tecorelabs.com` | `kotabonsai.com` |
| **Domain in VirtualService hosts** | `tecorelabs.com` | `kotabonsai.com` |
| **gateways field in VS** | `tecorelabs-gateway` | `kotabonsai-gateway` |
| **Path prefix** | `/kap/auth`, `/kap/org` | `/kap/auth`, `/kap/org` (same) |
| **DNS record** | `tecorelabs.com` → LB | `kotabonsai.com` → LB (same LB!) |

> **Important:** You can use the **same Istio Ingress Gateway** and **same Load Balancer** for multiple domains! Just create multiple Gateway resources.

---

## Multiple Domains on Same Cluster (Advanced)

```yaml
# gateway-tecorelabs.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: tecorelabs-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: tecorelabs-tls-secret
    hosts:
    - "tecorelabs.com"

---
# gateway-kotabonsai.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: kotabonsai-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: kotabonsai-tls-secret
    hosts:
    - "kotabonsai.com"
```

**Both use the SAME ingress gateway pod and SAME load balancer!** Istio's Envoy proxy reads the `Host` header and routes to the correct Gateway.

---

## Your New `values.yaml` for `kotabonsai.com`

```yaml
# values.yaml
domain: kotabonsai.com          # ← Changed from tecorelabs.com
project: kap
gateway: kotabonsai-gateway     # ← Changed from tecorelabs-gateway

services:
  - name: auth
    path: /auth
  - name: org
    path: /org
  - name: web
    path: /
  - name: datasource
    path: /datasource
  - name: kpi
    path: /kpi
  - name: projectdocs
    path: /projectdocs
```

And your Helm template:
```yaml
# templates/virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: {{ .Values.project }}-{{ .Values.name }}-route
  namespace: {{ .Release.Namespace }}
spec:
  hosts:
  - "{{ .Values.domain }}"
  gateways:
  - {{ .Values.gateway }}           # ← Uses kotabonsai-gateway!
  http:
  - match:
    - uri:
        prefix: /{{ .Values.project }}/{{ .Values.path }}
    route:
    - destination:
        host: {{ .Values.name }}-service
        port:
          number: 80
```

---

## Quick Checklist for New Domain Setup

| Step | Command / Action |
|------|----------------|
| 1. Install Istio | `istioctl install --set profile=demo -y` |
| 2. Get LB URL | `kubectl get svc istio-ingressgateway -n istio-system` |
| 3. Create TLS cert | cert-manager or manual secret |
| 4. Create Gateway | `kubectl apply -f gateway.yaml` |
| 5. Add DNS record | Point domain to LB URL |
| 6. Label namespaces | `kubectl label namespace <ns> istio-injection=enabled` |
| 7. Deploy apps + VirtualService | `helm install ...` |
| 8. Test | `curl https://kotabonsai.com/kap/auth` |

---

## What to Say in an Interview

> *"In my previous company, Istio was already set up by the platform team. I worked primarily with VirtualService resources to route traffic for our microservices. The platform team had created a Gateway resource in the istio-system namespace that accepted traffic for tecorelabs.com with a TLS certificate. My job was to create VirtualServices that linked to this Gateway via the `gateways` field and defined path-based routing like `/kap/auth` to the auth service. If I had to set this up from scratch for a new domain like kotabonsai.com, I would: install Istio with istioctl, get the ingress gateway load balancer URL, create a TLS certificate using cert-manager, create a new Gateway resource with the new domain and certificate, add a DNS record pointing to the LB, enable sidecar injection on application namespaces, and then deploy VirtualServices that reference the new Gateway."*

---

Does this clear up the confusion? Want me to show you how to debug if something goes wrong, or move on to anything else?
