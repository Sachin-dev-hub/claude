# Rapid-Fire Quick Answers — Practice Like Flashcards

**Cover the right column and say the answer out loud in one line.**

---

## Kubernetes

| Question | One-line answer |
|---|---|
| `kubectl get all -n dev` | Get all resources in the dev namespace |
| Check pod logs | `kubectl logs <pod> -n <ns>` (add `--previous` for crashed) |
| Rollback a deployment | `kubectl rollout undo deployment/<name>` |
| Drain a node safely | `kubectl drain <node> --ignore-daemonsets` |
| Scale a deployment | `kubectl scale deployment/<name> --replicas=5` |
| HPA basic command | `kubectl autoscale deployment/<name> --cpu-percent=70 --min=2 --max=8` |
| Check nodes health | `kubectl get nodes` |
| See events | `kubectl get events --sort-by=.lastTimestamp` |
| Pod stuck Pending | Not enough CPU/memory or no matching node |
| Pod CrashLoopBackOff | App crashing — check logs and config |
| Pod ImagePullBackOff | Wrong tag / registry auth / ECR login |
| Service not reachable | Check selector, `kubectl get endpoints`, DNS/CoreDNS |
| How DNS works in K8s | CoreDNS, service name = `svc.namespace.svc.cluster.local` |
| What stores cluster state | etcd (backup before upgrades!) |
| Liveness probe | Is app alive? Fail → restart |
| Readiness probe | Ready for traffic? Fail → remove from Service |
| Difference EKS vs self-managed | EKS = managed control plane, you manage nodes |
| Pod to S3 access on EKS | IRSA — IAM role on the ServiceAccount |
| HPA vs Cluster Autoscaler | HPA adds pods, CA adds nodes |

## Terraform

| Question | One-line answer |
|---|---|
| `terraform init` | Downloads providers and modules |
| `terraform plan` | Shows what will change (nothing applied) |
| `terraform apply` | Applies the changes |
| State locking uses | DynamoDB table |
| State storage best practice | S3 remote backend |
| Why state matters | Terraform needs it to know what it owns |
| Multiple environments | Workspaces OR separate state + tfvars per env |
| Reusable code | Modules (inputs = variables, outputs = values) |
| Manual change in AWS | Drift — `plan` shows it, `apply` fixes it |
| Terraform vs Ansible | Terraform = provision infra, Ansible = configure servers |

## Ansible

| Question | One-line answer |
|---|---|
| `ansible all -m ping` | Test Linux connection (SSH) |
| Windows connection protocol | WinRM (port 5986 HTTPS) |
| `ansible windows -m win_ping` | Test Windows connection |
| Run a playbook | `ansible-playbook site.yml -i inventory` |
| Dry run | `ansible-playbook site.yml --check` |
| Encrypt secrets | `ansible-vault encrypt secrets.yml` |
| Agentless means | Nothing installed on target servers |
| Idempotent means | Run many times → same result |
| Windows patching module | `win_updates` |
| Copy file to Windows | `win_copy` |

## Docker

| Question | One-line answer |
|---|---|
| Build an image | `docker build -t myapp:1.0 .` |
| Run a container | `docker run -p 8080:8080 myapp` |
| Push image | `docker push myapp:1.0` |
| Smaller images | Multi-stage builds |
| ENTRYPOINT vs CMD | ENTRYPOINT = main command, CMD = default args |
| Scan for vulns | `docker scan` / Trivy / ECR scanning |
| Docker vs K8s | Docker runs containers, K8s orchestrates them |

## AWS

| Question | One-line answer |
|---|---|
| ECR login | `aws ecr get-login-password \| docker login --username AWS --password-stdin <registry>` |
| EKS upgrade | `eksctl upgrade cluster --name <cluster>` |
| SG vs NACL | SG = instance, stateful; NACL = subnet, stateless |
| SQS vs SNS | SQS = queue (one consumer), SNS = pub/sub (many) |
| Multi-AZ RDS | For high availability / failover |
| Read replicas | For read scaling |
| Lambda limits | 15 min timeout, cold starts |
| HA VPC | 2+ AZs, private subnets + NAT, SGs |
| Presigned URL | Time-limited link to private S3 object |
| CloudWatch | AWS metrics + logs + alarms |

## CI/CD & Monitoring

| Question | One-line answer |
|---|---|
| CI | Build + test on every commit |
| CD | Deploy automatically (or with approval) |
| Build once, deploy many | Same image tag promoted across envs |
| Prod gate | Manual approval in Jenkins (`input`) |
| Blue-green | Two full versions, switch router |
| Canary | Small % traffic first, then ramp up |
| Golden signals | Latency, Traffic, Errors, Saturation |
| Prometheus | Collects metrics + alerts |
| Grafana | Visualizes dashboards |
| GitOps tools | Argo CD, Flux — Git is source of truth |

---

**Practice tip:** 10 cards at a time → say answer out loud → flip → repeat. Do this twice a day until interview.