Automated Multi-Tier Kubernetes Deployment Platform



An enterprise-grade GitOps platform deployed on a local multi-node Kubernetes cluster. The platform automates progressive application delivery using Canary rollouts, provisions persistent stateful storage, and maintains real-time cluster observability.



🏗️ Architecture Overview



graph TD

&#x20;   subgraph GitOps Control Plane

&#x20;       GH\[GitHub: devops-capstone-gitops]

&#x20;       ARGO\[ArgoCD GitOps Operator]

&#x20;   end



&#x20;   subgraph Cluster Controllers

&#x20;       ROLL\[Argo Rollouts Controller]

&#x20;       PROM\_OP\[Prometheus Operator]

&#x20;   end



&#x20;   subgraph Ingress Layer

&#x20;       ING\[NGINX Ingress Controller - Port 80]

&#x20;   end



&#x20;   subgraph Workload Tier: prod Namespace

&#x20;       FE\[Frontend Service - NGINX UI]

&#x20;       ACT\_SVC\[backend-active Service: 8080]

&#x20;       CAN\_SVC\[backend-canary Service: 8080]

&#x20;       RO\[Argo Rollout: backend-api<br/>Canary Traffic Weighting: 20% -> 50% -> 100%]

&#x20;       DB\[(PostgreSQL StatefulSet: 5432<br/>PVC: 1Gi Persistent Storage)]

&#x20;   end



&#x20;   subgraph Observability: monitoring Namespace

&#x20;       PROM\[Prometheus Server]

&#x20;       GRAF\[Grafana Dashboards]

&#x20;   end



&#x20;   GH -->|GitOps Reconcile| ARGO

&#x20;   ARGO -->|Syncs Declarative YAML| ING

&#x20;   ARGO -->|Syncs Declarative YAML| FE

&#x20;   ARGO -->|Syncs Declarative YAML| RO

&#x20;   ARGO -->|Syncs Declarative YAML| DB



&#x20;   ING -->|HTTP /| FE

&#x20;   FE -->|API Calls| ACT\_SVC

&#x20;   RO -.->|Traffic Weight Split| ACT\_SVC

&#x20;   RO -.->|Preview Traffic Split| CAN\_SVC

&#x20;   ACT\_SVC -->|Data Read/Write| DB



&#x20;   PROM\_OP -->|Scrapes Metrics| RO

&#x20;   PROM\_OP -->|Scrapes Metrics| FE

&#x20;   PROM\_OP -->|Scrapes Metrics| DB

&#x20;   PROM -->|Metrics Data Source| GRAF





💻 Tech Stack



Cluster Orchestration: Kubernetes (v1.30 multi-node Kind cluster: 1 Control Plane, 2 Worker Nodes)



GitOps Engine: ArgoCD



Progressive Delivery: Argo Rollouts (Canary Deployment Strategy)



Ingress \& Networking: NGINX Ingress Controller



Application Architecture: 3-Tier Multi-Tier Platform



Frontend: Static web server (NGINX Alpine)



Backend API: Argo Rollouts progressive Canary deployment



Database: PostgreSQL 15 StatefulSet with PersistentVolumeClaims (PVC)



Observability \& Monitoring: Prometheus \& Grafana (kube-prometheus-stack)



📂 Repository Layout



devops-capstone-gitops/

├── apps/

│   ├── database/

│   │   ├── postgres.yaml              # StatefulSet, Headless Service, PVC template

│   │   └── argocd-app-database.yaml   # ArgoCD App manifest

│   ├── backend/

│   │   ├── backend-rollout.yaml       # Argo Rollout with Canary traffic steps

│   │   └── argocd-app-backend.yaml    # ArgoCD App manifest

│   └── frontend/

│       ├── frontend.yaml              # Deployment, Service, and Ingress rule

│       └── argocd-app-frontend.yaml   # ArgoCD App manifest

└── infrastructure/

&#x20;   └── monitoring/

&#x20;       └── argocd-app-monitoring.yaml # Helm-driven kube-prometheus-stack App





⚙️ Key Technical Highlights



GitOps-Driven Reconciliation: All cluster resources are declaratively tracked in Git. Pushes to main are continuously reconciled by ArgoCD with automated drift detection and self-healing.



Zero-Downtime Canary Delivery: The backend uses Argo Rollouts to split traffic between stable and candidate versions (20% -> 50% -> 100%) before complete promotion.



Persistent Stateful Storage: PostgreSQL is configured using a StatefulSet with dynamic PersistentVolumeClaims to prevent data loss across pod rescheduling.



End-to-End Observability: Metrics across CPU, memory, node health, and pod states are captured by Prometheus and visualized in real time through Grafana.

