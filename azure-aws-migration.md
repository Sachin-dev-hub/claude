# Azure → AWS Migration — Interview Q&A

**JD explicitly asks: "Analyse existing Azure infrastructure and workloads and define appropriate AWS services and target architecture."**

---

## Q1. How would you map Azure services to AWS?

| Azure | AWS |
|---|---|
| Virtual Machine (VM) | EC2 |
| Virtual Network (VNet) | VPC |
| Blob Storage | S3 |
| Azure Files | EFS |
| AKS (Kubernetes) | EKS |
| Azure Functions | Lambda |
| SQL Database | RDS |
| App Service | ECS/EKS or Elastic Beanstalk |
| Service Bus Queue | SQS |
| Event Grid | EventBridge |
| Azure DevOps Pipelines | Jenkins / GitHub Actions / CodePipeline |
| Azure Data Lake | S3 (data lake) |
| Azure Synapse | Redshift / Athena + Glue |

---

## Q2. What migration strategies do you know? (the "6 Rs")

1. **Rehost** (lift & shift): move as-is, fastest, least change. VM → EC2 with same config.
2. **Replatform** (lift and shift + tweak): e.g., move to RDS instead of self-managed DB, move to EKS from VMs. Less work than refactor.
3. **Refactor / Re-architect:** rewrite for cloud-native (e.g., serverless with Lambda, containers on EKS). Most work, best long-term.
4. **Repurchase:** move to a different product (e.g., SaaS).
5. **Retire:** decommission applications nobody uses.
6. **Retain:** keep some workloads on Azure for now.

---

## Q3. What's your plan for a real migration project? (scenario question)

1. **Assess:** inventory all Azure workloads — what runs where, dependencies, data sizes.
2. **Map:** Azure service → AWS service (table above).
3. **Choose strategy per workload:** start with Rehost (quick wins), move to Replatform/Refactor for core apps.
4. **Migrate data:** copy to S3/RDS first (test data sync), then cut over.
5. **Set up AWS landing zone:** VPC, IAM, networking, security baseline first.
6. **CI/CD:** recreate pipelines in Jenkins/CodePipeline pointing to new AWS targets.
7. **Cutover + validate:** move traffic, run smoke tests, monitor.
8. **Operational readiness:** backups, monitoring, alerts, runbooks, on-call.
9. **Decommission** old Azure resources after a stable period.

---

## Q4. What is a "landing zone"?

- A **standard, secure, ready-made AWS environment** that follows best practices — before you migrate anything.
- Includes: multi-account setup (prod/dev/logging), VPCs, IAM roles, security baseline (CloudTrail, GuardDuty), monitoring.
- Purpose: so every new workload lands in a secure, consistent place instead of being built ad-hoc.

---

## Quick terms to remember

| Term | One-liner |
|---|---|
| 6 Rs | Rehost, Replatform, Refactor, Repurchase, Retire, Retain |
| Landing zone | Standard secure AWS foundation before migration |
| Cutover | The moment you switch traffic to the new environment |
| Lift & shift | Move as-is, fastest, least change |
| AWS DataSync / S3 Transfer | Data transfer tools for large data migration |