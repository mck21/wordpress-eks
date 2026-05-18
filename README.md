# WordPress on EKS + RDS + EFS

Deployment of WordPress on Amazon EKS using EFS for file persistence and RDS (MySQL) as the database backend.

## Architecture Overview

```
                        Internet
                           │
                    ┌──────▼──────┐
                    │     ALB     │  (created automatically by K8s LoadBalancer)
                    └──────┬──────┘
                           │
              ┌────────────▼────────────┐
              │        EKS Cluster       │
              │                          │
              │  ┌──────────────────┐    │
              │  │ wordpress pod x3 │    │
              │  └────────┬─────────┘    │
              └───────────┼──────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
     ┌────────▼────────┐   ┌──────────▼────────┐
     │   RDS MySQL      │   │    EFS Volume      │
     │  (wordpressdb)   │   │  /wordpress (html) │
     └──────────────────┘   └───────────────────┘
```

## Stack

| Component | Service |
|---|---|
| Container orchestration | Amazon EKS |
| Application | WordPress (official image) |
| Database | Amazon RDS MySQL (db.t3.micro) |
| File storage | Amazon EFS (Regional) |
| Load balancer | AWS ELB (Classic, auto-provisioned) |
| Secrets | Kubernetes Secret |

## Repository Structure

```
wordpress-eks/
├── README.md
├── docs/
│   ├── architecture.md      # Architecture description and design decisions
│   ├── step-by-step.md      # Full deployment guide
│   └── cost-estimate.md     # AWS cost breakdown
├── iac/
│   ├── 01-storageclass.yaml # EFS StorageClass
│   ├── 02-pv.yaml           # PersistentVolumes
│   ├── 03-pvc.yaml          # PersistentVolumeClaims
│   ├── 04-mariadb.yaml      # MariaDB deployment (phase 1 only)
│   ├── 05-wp-deployment.yaml# WordPress Service + Deployment
│   └── 06-rds-svc.yaml      # ExternalName Service pointing to RDS
└── images/                  # Screenshots and evidence
```

## Deployment Phases

The deployment is done in two phases:

**Phase 1 — MariaDB on Kubernetes:** WordPress + MariaDB both running as pods, sharing an EFS volume. Simple but not production-ready (EFS is not optimal for databases).

**Phase 2 — RDS migration:** MariaDB pod replaced by an RDS MySQL instance. WordPress pod remains unchanged; only a `Service/ExternalName` is swapped. This enables horizontal scaling.

## Quick Start

See [docs/step-by-step.md](docs/step-by-step.md) for the full deployment guide.

### Prerequisites

- EKS cluster running with worker nodes
- EC2 instance with `kubectl` configured
- EFS CSI Driver add-on installed on the cluster
- AWS CLI configured with valid session credentials

## Key Concepts

- **EFS Access Points** with POSIX UIDs matching container users (999 for MariaDB, 33 for WordPress) allow pods to create their directories on first run without manual intervention.
- **Kubernetes Secret** stores the DB password and is injected as an env var into both MariaDB and WordPress containers.
- **Service/ExternalName** allows pods to reach RDS using the same internal DNS name as the old MariaDB service — no changes to the WordPress manifest needed when migrating.
- **ReadWriteMany (RWX)** access mode on EFS PVCs allows multiple WordPress replicas to share the same `/var/www/html` directory simultaneously.
