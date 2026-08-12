# Cloud & Kubernetes — SRE Interview Guide

> **Target role:** HSBC SRE — Docker, Kubernetes, Istio, Helm, GitOps, multi-cloud  
> **Job description alignment:** AWS/Azure/GCP, containerized production, ArgoCD/FluxCD/Tekton

---

## Table of Contents

1. [Cloud Platform Overview](#cloud-platform-overview)
2. [Networking: VPC, Subnets, Security](#networking-vpc-subnets-security)
3. [Load Balancers & Autoscaling](#load-balancers--autoscaling)
4. [Docker Fundamentals](#docker-fundamentals)
5. [Kubernetes Fundamentals](#kubernetes-fundamentals)
6. [Kubernetes Objects](#kubernetes-objects)
7. [Helm, Istio, GitOps](#helm-istio-gitops)
8. [Deployment Strategies](#deployment-strategies)
9. [Production Scenarios](#production-scenarios)
10. [HSBC Cloud Expectations](#hsbc-cloud-expectations)

---

## Cloud Platform Overview

### Service Comparison (AWS / GCP / Azure)

| Category | AWS | GCP | Azure |
|----------|-----|-----|-------|
| **Compute (VM)** | EC2 | Compute Engine | Virtual Machines |
| **Containers** | ECS, EKS | GKE | AKS |
| **Serverless** | Lambda | Cloud Functions | Azure Functions |
| **Object Storage** | S3 | Cloud Storage | Blob Storage |
| **Block Storage** | EBS | Persistent Disk | Managed Disks |
| **Relational DB** | RDS | Cloud SQL | Azure SQL |
| **NoSQL** | DynamoDB | Firestore/Bigtable | Cosmos DB |
| **Load Balancer** | ALB/NLB/CLB | Cloud Load Balancing | Azure Load Balancer |
| **DNS** | Route 53 | Cloud DNS | Azure DNS |
| **IAM** | IAM | Cloud IAM | Azure AD/RBAC |
| **Secrets** | Secrets Manager | Secret Manager | Key Vault |
| **Monitoring** | CloudWatch | Cloud Monitoring | Azure Monitor |
| **Logging** | CloudWatch Logs | Cloud Logging | Log Analytics |

### HSBC Cloud Context

Global banks typically use **multi-cloud or hybrid** strategies:
- Regulatory requirements may mandate data residency (EU data in EU regions)
- Avoid single-vendor lock-in
- Different business units may use different clouds
- **Your role:** Support production on at least one of AWS/Azure/GCP with Kubernetes

---

## Networking: VPC, Subnets, Security

### VPC Architecture (AWS Example)

```
┌──────────────────────────────── VPC 10.0.0.0/16 ────────────────────────────────┐
│                                                                                  │
│  ┌─── AZ-a ──────────────┐    ┌─── AZ-b ──────────────┐                        │
│  │ Public Subnet          │    │ Public Subnet          │                        │
│  │ 10.0.1.0/24            │    │ 10.0.2.0/24            │                        │
│  │  ┌──────┐ ┌──────┐    │    │  ┌──────┐ ┌──────┐    │                        │
│  │  │ NAT  │ │ ALB  │    │    │  │ NAT  │ │ ALB  │    │                        │
│  │  └──┬───┘ └──┬───┘    │    │  └──┬───┘ └──┬───┘    │                        │
│  └─────┼────────┼─────────┘    └─────┼────────┼─────────┘                        │
│        │        │                    │        │                                  │
│  ┌─────▼────────▼─────────┐    ┌─────▼────────▼─────────┐                        │
│  │ Private Subnet          │    │ Private Subnet          │                        │
│  │ 10.0.10.0/24           │    │ 10.0.20.0/24           │                        │
│  │  ┌────────┐ ┌────────┐ │    │  ┌────────┐ ┌────────┐ │                        │
│  │  │ EKS    │ │ RDS    │ │    │  │ EKS    │ │ RDS    │ │                        │
│  │  │ Nodes  │ │Primary │ │    │  │ Nodes  │ │Replica │ │                        │
│  │  └────────┘ └────────┘ │    │  └────────┘ └────────┘ │                        │
│  └────────────────────────┘    └────────────────────────┘                        │
│                                                                                  │
│  Internet Gateway ◄──► Public Subnets                                           │
│  NAT Gateway ◄──► Private Subnets (outbound only)                               │
└──────────────────────────────────────────────────────────────────────────────────┘
```

### Key Networking Concepts

| Concept | Definition |
|---------|------------|
| **VPC** | Isolated virtual network |
| **Subnet** | IP range segment within VPC (tied to AZ) |
| **Route Table** | Rules for traffic routing |
| **Internet Gateway** | VPC ↔ Internet |
| **NAT Gateway** | Private subnet outbound internet |
| **Security Group** | Stateful firewall at instance/ENI level |
| **NACL** | Stateless firewall at subnet level |
| **Peering** | Connect two VPCs privately |
| **Transit Gateway** | Hub for many VPCs/on-prem |

### Security Groups vs NACLs

| | Security Group | NACL |
|---|----------------|------|
| **Level** | Instance/ENI | Subnet |
| **State** | Stateful (return traffic auto-allowed) | Stateless |
| **Rules** | Allow only | Allow + Deny |
| **Evaluation** | All rules evaluated | Ordered by rule number |
| **Default** | Deny all inbound | Allow all |

### Kubernetes Networking Overlay

```
Internet → ALB/Ingress Controller → Service (ClusterIP) → Pod
                                          │
                                    kube-proxy (iptables/IPVS)
                                          │
                                    CNI plugin (Calico, Cilium, AWS VPC CNI)
```

---

## Load Balancers & Autoscaling

### Cloud Load Balancer Types

| Type | Layer | Use Case |
|------|-------|----------|
| **ALB / Application LB** | L7 | HTTP routing, path-based, host-based |
| **NLB / Network LB** | L4 | TCP/UDP, extreme performance, static IP |
| **CLB / Classic LB** | L4/L7 | Legacy (avoid for new deployments) |
| **Gateway LB** | L3/L4 | Inline appliances (firewalls, IDS) |

### Autoscaling Patterns

```
┌─────────────────────────────────────────────────────────┐
│                  AUTOSCALING LAYERS                       │
│                                                           │
│  ┌─────────────┐  HPA scales pods based on CPU/memory/   │
│  │     HPA     │  custom metrics                          │
│  └──────┬──────┘                                          │
│         ▼                                                 │
│  ┌─────────────┐  Cluster Autoscaler adds/removes nodes  │
│  │ Cluster AS  │  when pods can't be scheduled            │
│  └──────┬──────┘                                          │
│         ▼                                                 │
│  ┌─────────────┐  Cloud ASG scales EC2/VM instances      │
│  │  Cloud ASG  │  based on demand                          │
│  └─────────────┘                                          │
└─────────────────────────────────────────────────────────┘
```

### HPA Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment-api-hpa
  namespace: payments
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
```

---

## Docker Fundamentals

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Image** | Immutable template (layers) |
| **Container** | Running instance of image |
| **Dockerfile** | Build instructions |
| **Registry** | Image storage (ECR, GCR, ACR, Docker Hub) |
| **Volume** | Persistent data storage |
| **Network** | Container networking (bridge, host, overlay) |

### Dockerfile Best Practices

```dockerfile
# Multi-stage build — smaller, more secure images
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /payment-api ./cmd/server

FROM gcr.io/distroless/static-debian12
COPY --from=builder /payment-api /payment-api
USER 65534:65534
EXPOSE 8080
ENTRYPOINT ["/payment-api"]
```

| Practice | Why |
|----------|-----|
| Multi-stage builds | Smaller final image, no build tools in prod |
| Non-root user | Security — limit container privileges |
| Pin base image versions | Reproducible builds |
| `.dockerignore` | Faster builds, no secrets in context |
| Distroless/minimal base | Reduced attack surface |
| HEALTHCHECK | Container orchestrator knows if app is healthy |

### Essential Docker Commands

```bash
# Build and run
docker build -t payment-api:v1.2.3 .
docker run -d -p 8080:8080 --name payment-api payment-api:v1.2.3

# Debug
docker logs payment-api -f --tail 100
docker exec -it payment-api /bin/sh
docker inspect payment-api
docker stats payment-api

# Cleanup
docker system prune -a       # remove unused images/containers
docker image ls --format "{{.Repository}}:{{.Tag}} {{.Size}}"
```

---

## Kubernetes Fundamentals

### Architecture

```
┌─────────────────── CONTROL PLANE ───────────────────┐
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │ API      │  │ etcd     │  │ Scheduler         │  │
│  │ Server   │  │ (state)  │  │ (pod placement)   │  │
│  └──────────┘  └──────────┘  └───────────────────┘  │
│  ┌──────────────────────────────────────────────┐    │
│  │ Controller Manager                            │    │
│  │ (Deployment, ReplicaSet, Node controllers)    │    │
│  └──────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
        ┌──────────┐┌──────────┐┌──────────┐
        │ Worker 1 ││ Worker 2 ││ Worker 3 │
        │ kubelet  ││ kubelet  ││ kubelet  │
        │ kube-    ││ kube-    ││ kube-    │
        │ proxy    ││ proxy    ││ proxy    │
        │ ┌──┐┌──┐││ ┌──┐┌──┐││ ┌──┐┌──┐│
        │ │P1││P2│││ │P3││P4│││ │P5││P6││
        │ └──┘└──┘││ └──┘└──┘││ └──┘└──┘│
        └──────────┘└──────────┘└──────────┘
```

### Key Components

| Component | Role |
|-----------|------|
| **API Server** | Front-end for Kubernetes API |
| **etcd** | Distributed key-value store (cluster state) |
| **Scheduler** | Assigns pods to nodes |
| **Controller Manager** | Reconciliation loops (desired vs actual state) |
| **kubelet** | Agent on each node, manages pods |
| **kube-proxy** | Network rules for Services |
| **CNI** | Container network interface plugin |

### Essential kubectl Commands

```bash
# Context and namespaces
kubectl config get-contexts
kubectl config use-context prod-cluster
kubectl get ns

# Workloads
kubectl get pods -n payments -o wide
kubectl describe pod <pod> -n payments
kubectl logs <pod> -n payments -f --previous
kubectl exec -it <pod> -n payments -- /bin/sh

# Deployments
kubectl rollout status deployment/payment-api -n payments
kubectl rollout history deployment/payment-api -n payments
kubectl rollout undo deployment/payment-api -n payments
kubectl scale deployment/payment-api --replicas=5 -n payments

# Debugging
kubectl get events -n payments --sort-by='.lastTimestamp'
kubectl top pods -n payments
kubectl top nodes
kubectl debug -it <pod> -n payments --image=busybox --target=<container>

# Resource management
kubectl apply -f deployment.yaml
kubectl diff -f deployment.yaml
kubectl delete -f deployment.yaml
```

---

## Kubernetes Objects

### Object Hierarchy

```
Namespace
  └── Deployment
        └── ReplicaSet
              └── Pod(s)
                    └── Container(s)
```

### Core Objects Explained

#### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-api
  namespace: payments
  labels:
    app: payment-api
    version: v1.2.3
spec:
  containers:
    - name: payment-api
      image: registry.company.com/payment-api:v1.2.3
      ports:
        - containerPort: 8080
      resources:
        requests:
          cpu: 250m
          memory: 256Mi
        limits:
          cpu: 500m
          memory: 512Mi
      livenessProbe:
        httpGet:
          path: /health
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
      envFrom:
        - configMapRef:
            name: payment-api-config
        - secretRef:
            name: payment-api-secrets
```

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
  namespace: payments
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: payment-api
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: payment-api
                topologyKey: topology.kubernetes.io/zone
      containers:
        - name: payment-api
          image: registry.company.com/payment-api:v1.2.3
          # ... (same as Pod spec above)
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-api
  namespace: payments
spec:
  selector:
    app: payment-api
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  type: ClusterIP    # ClusterIP | NodePort | LoadBalancer
```

#### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: payment-api-ingress
  namespace: payments
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.payments.company.com
      secretName: payment-api-tls
  rules:
    - host: api.payments.company.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: payment-api
                port:
                  number: 80
```

#### ConfigMap & Secret

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: payment-api-config
  namespace: payments
data:
  LOG_LEVEL: "info"
  DB_HOST: "postgres.payments.svc.cluster.local"
  DB_PORT: "5432"
---
apiVersion: v1
kind: Secret
metadata:
  name: payment-api-secrets
  namespace: payments
type: Opaque
stringData:
  DB_PASSWORD: "changeme"    # Use external secrets manager in prod!
  JWT_SECRET: "changeme"
```

### Object Comparison Table

| Object | Purpose | Manages |
|--------|---------|---------|
| **Pod** | Smallest deployable unit | Containers |
| **Deployment** | Declarative updates | ReplicaSets → Pods |
| **StatefulSet** | Stable identity, ordered | Pods with persistent storage |
| **DaemonSet** | One pod per node | Node-level agents (logging, monitoring) |
| **Job/CronJob** | Run-to-completion | Batch/scheduled tasks |
| **Service** | Stable network endpoint | Pod selection via labels |
| **Ingress** | External HTTP routing | Services |
| **ConfigMap** | Non-sensitive config | Key-value data |
| **Secret** | Sensitive data | Encrypted at rest |
| **PV/PVC** | Persistent storage | Disk volumes |
| **Namespace** | Logical isolation | All resources |
| **RBAC** | Access control | Roles, RoleBindings |
| **NetworkPolicy** | Pod-level firewall | Ingress/egress rules |
| **PDB** | Disruption budget | Min available during drains |

---

## Helm, Istio, GitOps

### Helm — Package Manager for Kubernetes

```
Chart Structure:
payment-api/
├── Chart.yaml          # Metadata
├── values.yaml         # Default config
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── charts/             # Dependencies
```

```bash
# Install/upgrade
helm install payment-api ./payment-api -n payments
helm upgrade payment-api ./payment-api -n payments --set image.tag=v1.2.4
helm rollback payment-api 3 -n payments

# Debug
helm template payment-api ./payment-api --debug
helm get values payment-api -n payments
```

### Istio — Service Mesh

```
┌─────────── Service Mesh Architecture ───────────┐
│                                                │
│  Pod A                    Pod B                │
│  ┌─────────────┐         ┌─────────────┐      │
│  │ App         │         │ App         │      │
│  │ ┌─────────┐ │         │ ┌─────────┐ │      │
│  │ │Sidecar  │◄├────────►│ │Sidecar  │ │      │
│  │ │(Envoy)  │ │  mTLS   │ │(Envoy)  │ │      │
│  │ └────┬────┘ │         │ └────┬────┘ │      │
│  └──────┼──────┘         └──────┼──────┘      │
│         │                       │              │
│         └───────┬───────────────┘              │
│                 ▼                               │
│         ┌──────────────┐                       │
│         │ Istio Control│                       │
│         │ Plane (istiod)│                       │
│         └──────────────┘                       │
└────────────────────────────────────────────────┘
```

| Istio Feature | SRE Benefit |
|---------------|-------------|
| **mTLS** | Encrypted service-to-service communication |
| **Traffic management** | Canary, A/B, fault injection |
| **Observability** | Automatic metrics, traces, access logs |
| **Circuit breaking** | Prevent cascading failures |

```yaml
# Istio VirtualService — Canary routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: payment-api
spec:
  hosts:
    - payment-api
  http:
    - route:
        - destination:
            host: payment-api
            subset: v1
          weight: 90
        - destination:
            host: payment-api
            subset: v2
          weight: 10
```

### GitOps — ArgoCD, FluxCD, Tekton

```
┌──────────── GitOps Flow ────────────────────────────┐
│                                                      │
│  Developer → Git Push → Git Repo (source of truth)  │
│                              │                       │
│                              ▼                       │
│                     ┌────────────────┐               │
│                     │ ArgoCD / Flux  │               │
│                     │ (reconciler)   │               │
│                     └───────┬────────┘               │
│                             │ desired state          │
│                             ▼                        │
│                     ┌────────────────┐               │
│                     │  Kubernetes  │               │
│                     │  Cluster     │               │
│                     └────────────────┘               │
│                                                      │
│  CI (Tekton/Jenkins/GitLab) → build/test → push     │
│  image + update manifest in Git                      │
└──────────────────────────────────────────────────────┘
```

| Tool | Role |
|------|------|
| **ArgoCD** | Continuous delivery — syncs Git → cluster |
| **FluxCD** | GitOps operator — alternative to ArgoCD |
| **Tekton** | Cloud-native CI/CD pipelines in Kubernetes |

**ArgoCD example Application:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payment-api
  namespace: argocd
spec:
  project: payments
  source:
    repoURL: https://git.company.com/platform/payment-api.git
    targetRevision: main
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: payments
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

**GitOps principles:**
1. **Declarative** — entire system described declaratively
2. **Versioned** — canonical config in Git
3. **Automated** — approved changes auto-applied
4. **Auditable** — Git history = audit trail

---

## Deployment Strategies

### Comparison

| Strategy | Downtime | Risk | Rollback Speed | Resource Cost |
|----------|----------|------|----------------|---------------|
| **Rolling** | None | Medium | Moderate | 1x |
| **Blue-Green** | None | Low | Instant (switch) | 2x |
| **Canary** | None | Very Low | Fast (shift traffic) | ~1.1x |
| **Recreate** | Yes | High | Slow | 1x |

### Rolling Update (Default Kubernetes)

```
v1 v1 v1  →  v1 v1 v2  →  v1 v2 v2  →  v2 v2 v2
              ↑ new pod     ↑ old removed
```

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # max extra pods during update
    maxUnavailable: 0    # zero downtime
```

### Blue-Green

```
                    ┌─────────┐
Traffic ──────────►│   LB    │
                    └────┬────┘
                         │
              ┌──────────┼──────────┐
              ▼                     ▼
        ┌──────────┐         ┌──────────┐
        │ Blue(v1) │         │Green(v2) │
        │ ACTIVE   │         │ STANDBY  │
        └──────────┘         └──────────┘

Switch: Update LB to point to Green → instant cutover
Rollback: Point LB back to Blue
```

### Canary

```
Traffic ──► LB ──► 95% → v1 (stable)
              └──►  5% → v2 (canary)

Monitor canary metrics → increase to 25% → 50% → 100%
If errors spike → route 100% back to v1
```

---

## Production Scenarios

### Scenario 1: Pod CrashLoopBackOff

```bash
kubectl describe pod payment-api-xyz -n payments
# Look at: Last State, Exit Code, Events

kubectl logs payment-api-xyz -n payments --previous

# Common causes:
# Exit 137 → OOMKilled (increase memory limits)
# Exit 1   → Application error (check logs)
# Exit 143 → SIGTERM (check probes/lifecycle)
```

### Scenario 2: Node Not Ready

```bash
kubectl get nodes
kubectl describe node ip-10-0-1-5

# Check kubelet on node
ssh node "systemctl status kubelet"
ssh node "journalctl -u kubelet -f"

# Common causes: disk pressure, memory pressure, network issue
kubectl describe node ip-10-0-1-5 | grep -A5 Conditions
```

### Scenario 3: Service Not Reachable

```bash
# DNS resolution inside cluster
kubectl run debug --rm -it --image=busybox -- nslookup payment-api.payments.svc.cluster.local

# Endpoints exist?
kubectl get endpoints payment-api -n payments

# Pod labels match service selector?
kubectl get pods -n payments --show-labels
kubectl get svc payment-api -n payments -o yaml | grep selector -A2

# NetworkPolicy blocking?
kubectl get networkpolicy -n payments
```

### Scenario 4: Certificate Expiry

```bash
# Check cert-manager certificates
kubectl get certificates -A
kubectl describe certificate payment-api-tls -n payments

# Manual check
echo | openssl s_client -connect api.payments.company.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## HSBC Cloud Expectations

### What Interviewers Likely Expect

| Area | Expectation |
|------|-------------|
| **Kubernetes** | Deploy, debug, scale containerized apps; understand probes, resources, RBAC |
| **GitOps** | ArgoCD/FluxCD workflow; Git as source of truth |
| **Monitoring** | Prometheus/Grafana in K8s; know ServiceMonitor, alerting rules |
| **Security** | NetworkPolicies, Pod Security Standards, secrets management |
| **IaC** | Terraform for cloud resources; Helm for K8s apps |
| **CI/CD** | Jenkins/GitLab CI building and deploying to K8s |
| **Service mesh** | Istio basics — mTLS, traffic management |
| **Multi-AZ** | Spread workloads across availability zones |
| **Compliance** | Audit trails, change management, least privilege |

### Production Readiness Checklist

```markdown
□ Resource requests/limits set
□ Liveness and readiness probes configured
□ Pod Disruption Budget defined
□ Horizontal Pod Autoscaler configured
□ NetworkPolicy applied
□ RBAC with least privilege
□ Secrets from external manager (not in Git)
□ Monitoring and alerting wired
□ Runbook documented
□ Rollback procedure tested
```

---

*Next: [04_ci_cd_automation.md](./04_ci_cd_automation.md)*
