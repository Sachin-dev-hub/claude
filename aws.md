# AWS — Interview Q&A

**Core services from the JD: S3, IAM, Lambda, EKS, RDS, VPC, Glue, CloudWatch, SQS/SNS, EventBridge.**

---

## Q1. How do you design a highly available AWS network (VPC)?

1. **VPC:** your private network.
2. **Availability Zones (AZs):** use at least 2–3 (e.g., `ap-south-1a`, `ap-south-1b`).
3. **Subnets:**
   - **Public subnets** — have Internet Gateway (IGW); host load balancers, NAT gateways, bastion.
   - **Private subnets** — no internet; host apps and databases.
4. **Route tables:** public subnet → IGW; private subnet → NAT gateway (outbound internet only).
5. **NAT gateway:** one per AZ (HA), lets private resources download packages, but nobody can reach them from outside.
6. **Security Groups (SG):** instance-level firewall (allow only needed ports, e.g., ALB → 80/443).
7. **NACL:** subnet-level firewall (stateless) — second layer.

**HA principle:** spread everything across multiple AZs so one AZ failing doesn't take down your app.

---

## Q2. Security Group vs NACL?

| | Security Group | NACL |
|---|---|---|
| Level | Instance | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow both directions) |
| Rules | Allow only | Allow and deny |
| Default | Deny all inbound | Allow all |

---

## Q3. How does IAM work? How do you enforce least privilege?

- IAM = Identity and Access Management.
- **Users** (people), **Groups** (collection of users), **Roles** (assumed by AWS services/EC2/pods), **Policies** (JSON — what actions on what resources).
- **Trust policy** = who can assume the role. **Permissions policy** = what they can do after assuming it.
- **Least privilege:** give only the exact permissions needed — start minimum, add.
- **Example:** instead of full S3 access for a pod, use IRSA — role with `s3:GetObject` on only `my-bucket/app-data/*`, attached to the pod's ServiceAccount.
- Use **IAM roles for EC2/ECS/EKS** instead of access keys (no keys to leak).

---

## Q4. S3 essentials?

- S3 = object storage. Store files: images, logs, data files, backups.
- **Key features:** versioning (protect from overwrite/delete), lifecycle rules (move old files to S3-IA/Glacier, then delete), bucket policies + IAM (access control), server-side encryption (SSE-S3/SSE-KMS), static website hosting.
- **Presigned URL:** temporary, time-limited link to a private object (e.g., download link valid 5 min).
- **S3 vs EBS vs EFS:**
  - **EBS:** block storage attached to one EC2 (like a hard disk) — for databases.
  - **EFS:** shared file storage (NFS) across many EC2s — shared files.
  - **S3:** object storage, unlimited scale — files, backups, data lake.

---

## Q5. What is a Lambda and how do you use it?

- Lambda = **serverless** compute. Upload code, AWS runs it. No servers to manage.
- **Triggers:** S3 event (file uploaded → process it), API Gateway (HTTP → run code), SQS, EventBridge (scheduled/rule), CloudWatch events.
- **Limits:** 15 min max timeout, 1 GB ephemeral storage, cold starts (first call slower).
- **Use cases in a data platform:** process a file when it lands in S3, trigger a job, scheduled cleanup.
- **Lambda vs EKS:** Lambda for small event-driven tasks; EKS for long-running, stateful, complex workloads.

---

## Q6. CloudWatch — what can it monitor?

- **Metrics:** CPU, memory, disk, network for EC2; custom metrics from your apps.
- **Logs:** CloudWatch Logs — collect app logs, search, metric filters, trigger alarms.
- **Alarms:** e.g., "CPU > 80% for 5 min" → SNS → email/PagerDuty.
- **Dashboards:** visual view of metrics.
- **CloudWatch vs Prometheus/Grafana:** CloudWatch is AWS-native (easy). Prometheus + Grafana are better for Kubernetes (pod-level metrics, richer dashboards). In EKS you often run both.

---

## Q7. SQS vs SNS vs EventBridge?

| Service | Type | Use case |
|---|---|---|
| **SQS** | Message **queue** (one consumer takes the message) | Decouple workers, buffer load, data ingestion |
| **SNS** | **Pub/sub** (message to many subscribers) | Notifications: email, SMS, Lambda, SQS |
| **EventBridge** | **Event bus** + rules + schedules | "When X happens, do Y"; scheduled jobs |

**Memory trick:** SQS = one person eats the food. SNS = broadcast to everyone. EventBridge = if/then rules.

---

## Q8. RDS — how do you make a database highly available?

- **Multi-AZ:** synchronous replica in another AZ → automatic failover. For **high availability**.
- **Read Replicas:** async copies for **read scaling** (offload SELECTs) and DR.
- **Backups:** automated daily snapshots + transaction logs (point-in-time restore), manual snapshots.
- **RDS vs own DB:** RDS handles patching, backups, failover for you (managed).

---

## Q9. ECS vs EKS — when to choose which?

- **ECS:** AWS's simpler container service. Less setup, deeply integrated (ALB, IAM, CloudWatch). For simplicity/smaller teams.
- **EKS:** full Kubernetes — portable, rich ecosystem (Helm, ArgoCD, Istio). For complex/multi-cloud needs. **The JD requires EKS production experience.**
- **Good answer:** "I use ECS for simple quick services and EKS for the main platform because of portability and ecosystem."

---

## Q10. How do you give a pod in EKS access to S3? (IRSA)

1. Create an IAM role with S3 read policy + trust policy allowing `sts:AssumeRoleWithWebIdentity` for the EKS cluster's OIDC provider.
2. Annotate the ServiceAccount: `eks.amazonaws.com/role-arn: arn:aws:iam::...:role/my-s3-role`.
3. Pods using that ServiceAccount get temporary S3 credentials automatically. No access keys in the pod.

---

## Quick commands to remember

| Command | What it does |
|---|---|
| `aws s3 ls s3://bucket` | List S3 bucket |
| `aws ecr get-login-password \| docker login --username AWS --password-stdin <registry>` | ECR docker login |
| `aws ec2 describe-instances` | List EC2 instances |
| `aws cloudwatch describe-alarms` | List CloudWatch alarms |
| `eksctl upgrade cluster --name <cluster>` | Upgrade EKS cluster |