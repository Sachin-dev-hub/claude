# Kubernetes & EKS — Interview Q&A

**JD requires production EKS experience — this is the most important topic.**

---

## Q1. How do you upgrade a Kubernetes cluster?

**Short answer:** Upgrade the control plane first, then worker nodes one at a time (so pods keep running). Always go one minor version at a time.

**For a self-managed (kubeadm) cluster:**
1. Check current version: `kubectl get nodes`
2. Check available versions: `kubeadm upgrade plan`
3. **Backup etcd first** (most important step).
4. Upgrade control plane (on master node):
   - Upgrade `kubeadm`: `apt upgrade kubeadm` (or `yum`)
   - `kubeadm upgrade apply v1.28.0`
   - Upgrade `kubectl` and `kubelet`, then `systemctl restart kubelet`
5. Upgrade worker nodes **one at a time**:
   - `kubectl drain node-1 --ignore-daemonsets` (moves pods to other nodes)
   - Upgrade kubeadm, kubelet, kubectl on that node
   - `kubeadm upgrade node`
   - `systemctl restart kubelet`
   - `kubectl uncordon node-1` (allow pods back)
6. Verify workloads: `kubectl get pods -A`

**Why one node at a time?** Draining keeps your application available — zero downtime.

**For EKS (managed cluster):**
1. Control plane is managed by AWS — trigger the update:
   - `eksctl upgrade cluster --name my-cluster` (or AWS Console → EKS → Upgrade)
2. Update node groups:
   - `eksctl upgrade nodegroup --cluster my-cluster --name my-nodegroup`
   - Or update the AMI / launch template and roll nodes.
3. Update add-ons (CoreDNS, kube-proxy, VPC CNI): `eksctl upgrade addon`
4. Verify apps after the upgrade.

**Interview tip:** Say "backup etcd before upgrade" and "drain nodes one at a time" — these show real production experience.

---

## Q2. How do you upgrade an application (deployment) in Kubernetes?

**Short answer:** Push a new image, then let Kubernetes do a rolling update. Rollback is one command.

**Steps:**
1. Build a new image and push to registry (e.g., `myapp:v2`)
2. Update the deployment:
   - `kubectl set image deployment/myapp myapp=myapp:v2 --record`
   - Or edit YAML and `kubectl apply -f deployment.yaml`
3. Watch the roll: `kubectl rollout status deployment/myapp`
4. Check history: `kubectl rollout history deployment/myapp`
5. If wrong, rollback: `kubectl rollout undo deployment/myapp`

**Deployment strategies:**
- **Rolling update (default):** replaces pods gradually. Tuned with `maxSurge` (extra pods during update) and `maxUnavailable` (pods allowed down). Example: `maxSurge: 1`, `maxUnavailable: 0` = zero downtime.
- **Recreate:** stop all old pods, then start new ones (downtime — only for non-critical apps).
- **Blue-green:** deploy new version alongside old, switch the Service selector to new. Rollback = switch selector back.
- **Canary:** deploy 10% new version, watch metrics, then roll to 100%.

---

## Q3. What happens when you run `kubectl apply -f deployment.yaml`?

- `kubectl` sends the **desired state** (your YAML) to the API server.
- Kubernetes stores it in **etcd**.
- The **Deployment controller** sees desired vs current state.
- It creates a **ReplicaSet**, which creates **Pods**.
- The **kubelet** on each worker node pulls the image and starts containers.
- This is **declarative** — you say WHAT you want, Kubernetes figures out HOW.

---

## Q4. Deployment vs StatefulSet vs DaemonSet vs Job vs CronJob?

| Type | Use for | Example |
|---|---|---|
| **Deployment** | Stateless apps, scale freely | Web/API apps |
| **StatefulSet** | Stable identity, stable storage, ordering | Databases, Kafka, Cassandra |
| **DaemonSet** | One pod on EVERY node | Fluentd (logs), Node Exporter (metrics), VPC CNI |
| **Job** | Run once and exit | Batch processing |
| **CronJob** | Run on a schedule | Nightly cleanup, backups |

---

## Q5. Service types and Ingress — difference?

- **ClusterIP (default):** reachable only inside the cluster.
- **NodePort:** opens a port on every node (30000–32767), reachable from outside via `nodeIP:port`.
- **LoadBalancer:** creates a cloud LB (AWS NLB/ALB) that forwards to the Service.
- **ExternalName:** returns a DNS record pointing outside the cluster.

**Ingress:** a smart "front door" — routing by hostname/path, TLS/HTTPS termination, sends traffic to different Services. One Ingress serves many apps (saves cost of many LoadBalancers).
Example: `api.mycompany.com` → service A, `app.mycompany.com` → service B.

---

## Q6. ConfigMap vs Secret — how to use them?

- **ConfigMap:** non-sensitive config (env vars, config files). Not encrypted.
- **Secret:** sensitive data (passwords, keys). Stored as base64 (obfuscated, not truly encrypted by default).
- **Inject as:** environment variables (`envFrom`) or files (mounted volumes).
- **Best practice:** keep config/secrets OUT of the image, inject at runtime. One image works in dev/stage/prod by changing config.
- **For EKS, better:** AWS Secrets Manager CSI driver or External Secrets Operator — secrets never sit inside etcd.

---

## Q7. How does autoscaling work?

**Two levels:**
1. **HPA (Horizontal Pod Autoscaler)** — scales the NUMBER of pods.
   - Uses **Metrics Server** to read CPU/memory.
   - `kubectl autoscale deployment myapp --cpu-percent=70 --min=3 --max=10`
   - Over 70% CPU → HPA adds pods (up to 10).
2. **Cluster Autoscaler / Karpenter** — scales the NUMBER of NODES.
   - Pods Pending (no node space) → CA adds a node.
   - Karpenter = newer AWS tool, faster and cheaper, adds nodes in seconds.

**One line:** HPA adds pods, Cluster Autoscaler adds nodes, Karpenter adds nodes faster and smarter.

---

## Q8. How do you troubleshoot a failing pod?

**Step-by-step flow (favorite interview answer):**
1. `kubectl get pods` — see the STATUS.
2. `kubectl describe pod <name>` — Events at the bottom tell you the real reason.
3. `kubectl logs <name>` — application errors.
4. `kubectl logs <name> --previous` — logs from last crashed container.

**Common statuses and fixes:**

| Status | Cause | Fix |
|---|---|---|
| **Pending** | Not enough CPU/mem, tainted node, no matching node | Check requests/limits, remove taint, add node |
| **ImagePullBackOff / ErrImagePull** | Wrong tag, private registry without creds, ECR auth missing | Fix tag, add `imagePullSecrets`, ECR login |
| **CrashLoopBackOff** | App starting and crashing (bad config, missing env) | Check logs, ConfigMap/Secret, fix app |
| **Node NotReady** | Node down, kubelet stopped, disk full | `kubectl get nodes`, check node, restart kubelet |
| **Service not reachable** | DNS/kube-proxy issue, wrong selector | Check selector matches labels, `kubectl get endpoints` |

---

## Q9. How does DNS work inside Kubernetes?

- **CoreDNS** runs as pods, is the cluster DNS server.
- Every Service gets a name: `service-name.namespace.svc.cluster.local`.
- Pods reach `myapp` (same ns) or `myapp.other-ns` (cross ns).
- `nslookup myapp` fails → CoreDNS issue → check CoreDNS pods and config.

---

## Q10. What is etcd and why is it important?

- etcd is the **brain** of Kubernetes — stores all cluster state (resources, config, secrets).
- If etcd is lost, the cluster is lost.
- **Backup regularly:** `etcdctl snapshot save` (EKS snapshots via AWS).
- Restore: `etcdctl snapshot restore`.
- Always back up etcd **before any cluster upgrade**.

---

## Q11. Namespaces — how do you isolate environments in one cluster?

- A Namespace is a virtual cluster inside one cluster.
- Separate teams/environments: `dev`, `staging`, `prod`.
- **ResourceQuota** limits total CPU/memory per namespace; **LimitRange** sets default per pod.
- **Network isolation** needs a Network Policy (e.g., prod can't talk to dev).
- `kubectl get ns`, `kubectl -n dev get pods`.

---

## Q12. What is RBAC in Kubernetes?

- RBAC = Role Based Access Control — decides WHO can do WHAT.
- **Role:** permissions in one namespace.
- **ClusterRole:** permissions across the whole cluster.
- **RoleBinding / ClusterRoleBinding:** connects a Role to a user or ServiceAccount.
- Example: give the CI/CD pipeline a ServiceAccount that can only deploy in `staging`.
- This enforces **least privilege** in Kubernetes.

---

## Q13. What is special about EKS vs a normal Kubernetes cluster?

- EKS = **managed control plane**. AWS runs/maintains the API server and etcd (HA across AZs).
- You only manage **worker nodes** (node groups) and workloads.
- **IAM integration:** nodes assume IAM roles. **IRSA (IAM Roles for Service Accounts)** lets each pod assume its own IAM role — pod can access S3 without hard-coded keys.
- Uses **VPC CNI** — pods get real VPC IPs, so AWS security groups, VPC flow logs, ALB/NLB work directly with pods.
- Updates are easier than self-managed (no etcd backup needed).

---

## Q14. Taints, Tolerations, Node Selectors, Affinity?

- **Node Selector:** simple — put pods only on nodes with a label (e.g., GPU pods → nodes labeled `gpu=true`).
- **Taints + Tolerations:** taint on a node says "no pod here unless it tolerates me". Keeps dedicated nodes (e.g., monitoring pods on monitoring nodes).
- **Affinity:** advanced rules — "prefer AZ-a" (soft) or "must be AZ-a" (hard); pod affinity (keep together) or anti-affinity (spread apart).

---

## Q15. Liveness vs Readiness vs Startup probes?

- **Liveness:** is the app alive? If it fails, Kubernetes **restarts** the container.
- **Readiness:** is the app ready to serve traffic? If it fails, pod is removed from the Service (no traffic, no restart).
- **Startup:** for slow-starting apps — gives time to start before liveness kicks in (prevents restarts during startup).

**Example:** a web app may be alive (liveness OK) but still loading config (readiness fails) → no traffic sent until ready.

---

## Quick commands to remember

| Command | What it does |
|---|---|
| `kubectl get all -n dev` | All resources in dev namespace |
| `kubectl logs <pod> --previous` | Logs from last crashed container |
| `kubectl rollout undo deployment/<name>` | Rollback a deployment |
| `kubectl drain <node> --ignore-daemonsets` | Drain a node safely |
| `kubectl scale deployment/<name> --replicas=5` | Scale a deployment |
| `kubectl get events --sort-by=.lastTimestamp` | See events |
| `eksctl upgrade cluster --name <cluster>` | Upgrade EKS cluster |