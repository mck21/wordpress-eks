# AWS Cost Estimate

Estimated costs for running this lab in `us-east-1`. All prices are approximate and based on on-demand rates as of 2026. AWS Academy lab credits apply.

---

## Resources and hourly cost

| Service | Resource | Spec | $/hour |
|---|---|---|---|
| EKS | Cluster control plane | — | $0.10 |
| EC2 | Worker node x2 | t3.medium | $0.052 x2 |
| EC2 | vscode instance | t2.micro | $0.0116 |
| RDS | MySQL instance | db.t3.micro | $0.017 |
| EFS | File system | ~0.1 GB used | ~$0.00 |
| ELB | Classic load balancer | 1 LB | $0.025 |
| **Total** | | | **~$0.27/hour** |

---

## Cost by phase

### Phase 1 (EKS + EFS + MariaDB pod)
No RDS instance running. Estimated ~**$0.25/hour**.

### Phase 2 (EKS + EFS + RDS)
RDS instance added. Estimated ~**$0.27/hour**.

---

## Full lab estimate

| Duration | Estimated cost |
|---|---|
| 3 hours (active lab) | ~$0.80 |
| 8 hours (left running) | ~$2.15 |
| 24 hours (forgot to clean up) | ~$6.45 |

---

## Cost reduction tips

- **Delete worker nodes when not in use.** The EKS control plane costs $0.10/h regardless, but stopping the node group eliminates EC2 costs.
- **Delete RDS when done.** Even stopped RDS instances accrue storage costs after 7 days.
- **EFS costs are negligible** at lab scale (< 1 GB used).
- **The ALB** is created automatically by Kubernetes and deleted when the LoadBalancer Service is deleted. Don't forget to delete the Service before destroying the cluster or the LB may become an orphan.

---

## AWS Academy note

This lab runs within AWS Academy, where session credentials expire every few hours. Resources persist between sessions but `~/.aws/credentials` must be refreshed at the start of each session:

```bash
aws configure
# or
nano ~/.aws/credentials
```
