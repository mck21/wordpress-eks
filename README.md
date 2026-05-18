# WordPress on EKS + RDS + EFS

Deployment of WordPress on Amazon EKS using EFS for file persistence and RDS (MySQL) as the database backend.

![Architecture diagram](images/architecture.png)

Full architecture explanation: [docs/architecture.md](docs/architecture.md)

## Stack

| Component | Service |
|---|---|
| Container orchestration | ![Amazon EKS](https://img.shields.io/badge/Amazon%20EKS-FF9900?style=flat&logo=amazoneks&logoColor=white) |
| Application | ![WordPress](https://img.shields.io/badge/WordPress-21759B?style=flat&logo=wordpress&logoColor=white) |
| Database | ![Amazon RDS](https://img.shields.io/badge/Amazon%20RDS-527FFF?style=flat&logo=amazonrds&logoColor=white) MySQL db.t3.micro |
| File storage | ![Amazon EFS](https://img.shields.io/badge/Amazon%20EFS-FF9900?style=flat&logo=amazonaws&logoColor=white) Regional |
| Load balancer | ![AWS ELB](https://img.shields.io/badge/AWS%20ELB-FF9900?style=flat&logo=amazonaws&logoColor=white) Classic, auto-provisioned |
| Secrets | ![Kubernetes](https://img.shields.io/badge/Kubernetes%20Secret-326CE5?style=flat&logo=kubernetes&logoColor=white) |

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
└── images/                  # Screenshots 
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
