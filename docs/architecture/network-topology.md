# Network Topology — GovCloud Environment

This page documents the fictional network architecture for our OpenShift clusters running in AWS GovCloud, using Mermaid diagrams to visualize the topology.

## High-Level Network Overview

```mermaid
graph TB
    Internet["Internet / Public Users"]
    
    subgraph AWS_GovCloud["AWS GovCloud (us-gov-west-1)"]
        subgraph VPC_Prod["Production VPC (10.0.0.0/16)"]
            subgraph PubSub["Public Subnets"]
                ALB["Application Load Balancer<br/>api.example.gov"]
                NLB["Network Load Balancer<br/>*.apps.example.gov"]
            end
            
            subgraph PrivSub["Private Subnets"]
                subgraph OCP_Prod["OpenShift Cluster — prd-cluster-01"]
                    Master1["Control Plane<br/>m1 (m5.2xlarge)"]
                    Master2["Control Plane<br/>m2 (m5.2xlarge)"]
                    Master3["Control Plane<br/>m3 (m5.2xlarge)"]
                    Worker1["Worker Node<br/>w1 (m5.4xlarge)"]
                    Worker2["Worker Node<br/>w2 (m5.4xlarge)"]
                    Worker3["Worker Node<br/>w3 (m5.4xlarge)"]
                    Infra1["Infra Node<br/>i1 (r5.2xlarge)"]
                    Infra2["Infra Node<br/>i2 (r5.2xlarge)"]
                end
            end
            
            subgraph DataSub["Data Subnets"]
                RDS["RDS PostgreSQL<br/>Multi-AZ"]
                ElastiCache["ElastiCache Redis<br/>Cluster Mode"]
                S3["S3 Bucket<br/>app-data-prod"]
            end
        end
        
        TGW["Transit Gateway"]
        
        subgraph VPC_Mgmt["Management VPC (10.1.0.0/16)"]
            Bastion["Bastion Host"]
            Vault["HashiCorp Vault"]
            Monitoring["Monitoring Stack<br/>Prometheus / Grafana"]
        end
    end
    
    Internet --> ALB
    Internet --> NLB
    ALB --> Worker1
    ALB --> Worker2
    ALB --> Worker3
    NLB --> Infra1
    NLB --> Infra2
    Worker1 --> RDS
    Worker2 --> ElastiCache
    Worker3 --> S3
    VPC_Prod --- TGW
    TGW --- VPC_Mgmt
    Bastion --> Master1
    Monitoring --> Infra1
```

## Service Mesh Traffic Flow

This diagram shows how requests flow through the service mesh within the OpenShift cluster.

```mermaid
flowchart LR
    Client["Client Request"]
    
    subgraph Ingress["Ingress Layer"]
        Router["OpenShift Router<br/>(HAProxy)"]
    end
    
    subgraph Mesh["Service Mesh (Istio)"]
        GW["Istio Gateway"]
        VS["Virtual Service"]
        
        subgraph SvcA["order-service"]
            SvcA_Proxy["Envoy Sidecar"]
            SvcA_App["App Container<br/>:8080"]
        end
        
        subgraph SvcB["inventory-service"]
            SvcB_Proxy["Envoy Sidecar"]
            SvcB_App["App Container<br/>:8080"]
        end
        
        subgraph SvcC["notification-service"]
            SvcC_Proxy["Envoy Sidecar"]
            SvcC_App["App Container<br/>:8080"]
        end
    end
    
    subgraph Data["Data Layer"]
        DB[(PostgreSQL)]
        Cache[(Redis)]
        Queue[(Kafka)]
    end
    
    Client --> Router --> GW --> VS
    VS --> SvcA_Proxy --> SvcA_App
    SvcA_Proxy --> SvcB_Proxy --> SvcB_App
    SvcA_Proxy --> SvcC_Proxy --> SvcC_App
    SvcA_App --> DB
    SvcB_App --> Cache
    SvcC_App --> Queue
```

## Deployment Pipeline

The CI/CD pipeline from code commit to production deployment.

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant CI as CI Pipeline
    participant Quay as Quay.io Registry
    participant AI as app-interface
    participant OCP as OpenShift

    Dev->>GH: Push commit to feature branch
    Dev->>GH: Open Pull Request
    GH->>CI: Trigger CI build
    CI->>CI: Run unit tests
    CI->>CI: Run linter & SAST scan
    CI->>CI: Build container image
    CI->>Quay: Push image (tag: sha-abc123)
    CI->>GH: Report status ✓
    Dev->>GH: Merge PR to main
    GH->>CI: Trigger release build
    CI->>Quay: Push image (tag: v1.2.3)
    Dev->>AI: Update SaaS file with new tag
    AI->>AI: Validate schema & run checks
    AI->>OCP: Reconcile — apply manifests
    OCP->>OCP: Rolling update deployment
    OCP-->>Dev: Deployment complete
```

## Cluster Node Architecture

```mermaid
graph TB
    subgraph Cluster["prd-cluster-01"]
        subgraph CP["Control Plane Nodes (3x m5.2xlarge)"]
            etcd["etcd"]
            API["API Server"]
            Scheduler["Scheduler"]
            Controller["Controller Manager"]
        end
        
        subgraph Infra["Infrastructure Nodes (2x r5.2xlarge)"]
            Router2["OpenShift Router"]
            Prom["Prometheus"]
            ES["Elasticsearch"]
            Grafana2["Grafana"]
        end
        
        subgraph Workers["Worker Nodes (3x m5.4xlarge)"]
            subgraph NS1["order-service--prod"]
                Pod1A["order-svc<br/>replica 1"]
                Pod1B["order-svc<br/>replica 2"]
                Pod1C["order-svc<br/>replica 3"]
            end
            
            subgraph NS2["inventory-service--prod"]
                Pod2A["inventory-svc<br/>replica 1"]
                Pod2B["inventory-svc<br/>replica 2"]
            end
            
            subgraph NS3["notification-service--prod"]
                Pod3A["notif-svc<br/>replica 1"]
                Pod3B["notif-svc<br/>replica 2"]
            end
        end
    end
    
    API --> Workers
    API --> Infra
    Prom --> NS1
    Prom --> NS2
    Prom --> NS3
```

## Disaster Recovery Architecture

This shows the failover setup between primary and DR regions.

```mermaid
flowchart TB
    DNS["Route 53<br/>Weighted DNS"]
    
    subgraph Primary["us-gov-west-1 (Primary)"]
        P_LB["Load Balancer"]
        P_OCP["prd-cluster-01<br/>Active"]
        P_RDS["RDS Primary"]
        P_S3["S3 Primary"]
    end
    
    subgraph DR["us-gov-east-1 (DR)"]
        DR_LB["Load Balancer"]
        DR_OCP["prd-cluster-02<br/>Standby"]
        DR_RDS["RDS Read Replica"]
        DR_S3["S3 Replica"]
    end
    
    DNS -->|"90% traffic"| P_LB
    DNS -->|"10% traffic"| DR_LB
    P_LB --> P_OCP
    DR_LB --> DR_OCP
    P_OCP --> P_RDS
    DR_OCP --> DR_RDS
    P_RDS -->|"Async replication"| DR_RDS
    P_S3 -->|"Cross-region replication"| DR_S3
    P_OCP --> P_S3
    DR_OCP --> DR_S3
```

## Monitoring & Alerting Flow

```mermaid
flowchart LR
    subgraph Sources["Metric Sources"]
        App["Application Pods"]
        Node["Node Exporters"]
        OCP["OpenShift Metrics"]
    end
    
    subgraph Collection["Collection"]
        Prom2["Prometheus"]
        Loki["Loki<br/>(Logs)"]
    end
    
    subgraph Visualization["Visualization"]
        Graf["Grafana<br/>Dashboards"]
    end
    
    subgraph Alerting["Alerting"]
        AM["AlertManager"]
        PD["PagerDuty"]
        Slack["Slack<br/>#sre-alerts"]
        Jira["Jira<br/>Auto-create tickets"]
    end
    
    App --> Prom2
    Node --> Prom2
    OCP --> Prom2
    App --> Loki
    Prom2 --> Graf
    Loki --> Graf
    Prom2 --> AM
    AM --> PD
    AM --> Slack
    AM --> Jira
```

## Network Security Zones

```mermaid
graph TB
    subgraph Zone0["Zone 0 — Internet"]
        ExtUser["External Users"]
    end
    
    subgraph Zone1["Zone 1 — DMZ"]
        WAF["AWS WAF"]
        CF["CloudFront CDN"]
    end
    
    subgraph Zone2["Zone 2 — Application"]
        LB2["Load Balancer"]
        OCP2["OpenShift Cluster"]
    end
    
    subgraph Zone3["Zone 3 — Data"]
        DB2["Databases"]
        Cache2["Cache"]
        ObjectStore["Object Storage"]
    end
    
    subgraph Zone4["Zone 4 — Management"]
        SIEM["SIEM"]
        Bastion2["Bastion"]
        Vault2["Vault"]
    end
    
    ExtUser -->|"HTTPS 443"| WAF
    WAF -->|"HTTPS 443"| CF
    CF -->|"HTTPS 443"| LB2
    LB2 -->|"HTTP 8080"| OCP2
    OCP2 -->|"TCP 5432"| DB2
    OCP2 -->|"TCP 6379"| Cache2
    OCP2 -->|"HTTPS 443"| ObjectStore
    Zone4 -.->|"Audit Logs"| SIEM
    Bastion2 -.->|"SSH 22"| OCP2
    Vault2 -.->|"HTTPS 8200"| OCP2
```

## Related Pages

- [Architecture Overview](overview.md) — system components and data flow
- [Deployment Architecture](deployment.md) — CI/CD and scaling
- [Incident Response](../runbooks/incident-response.md) — what to do when things break
