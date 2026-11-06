# CI/CD & Jenkins — Interview Q&A

**Includes the KEY question: "How do you manage a CI/CD pipeline in multiple environments?"**

---

## Q1. What is CI and what is CD?

- **CI (Continuous Integration):** every code commit is automatically built and tested. Catches bugs early.
- **CD (Continuous Delivery/Deployment):** after CI passes, automatically (or with approval) deploy to environments, up to production.
- **Your pipeline (from resume):** GitHub → Jenkins → Maven build → SonarQube (static code analysis) → Docker image → push to ECR → deploy to EKS.

---

## Q2. How do you manage a CI/CD pipeline across multiple environments (dev, staging, prod)? (KEY — practice this well)

**The core idea: build ONCE, deploy MANY times. Same pipeline, environment as a parameter.**

1. **Single pipeline, parameterized:** one Jenkinsfile (or one pipeline) that takes `ENV` as a parameter (`dev`, `staging`, `prod`). Stages are the same; only the target changes.
2. **Promote the same artifact, never rebuild:** build and test once → push the image with a tag like `myapp-1.2.3-<commit>`. Dev, staging, prod all deploy THIS SAME image. If prod works, it works everywhere because it's the same artifact.
3. **Environment-specific config:** the image is identical; differences live OUTSIDE the image — ConfigMaps/Secrets/Helm values per environment. This is why we separate config from the image.
4. **Gates between environments:**
   - Dev: auto-deploy on every merge to develop.
   - Staging: deploy on tag/release; run integration + SonarQube quality gate; fail pipeline if quality gate fails.
   - Prod: **manual approval** step (`input` in Jenkins) — only a senior person clicks "Approve". Deploy the exact same tag that passed staging.
5. **Environment-specific credentials:** separate credentials in Jenkins per env (prod credentials only used in prod stages), or fetch from Vault/Secrets Manager.
6. **Deployment strategy per env:** rolling update for dev/staging; blue-green or canary with monitoring for prod (so rollback is instant).
7. **Rollback plan:** keep the previous image tag, one command to roll back (`kubectl rollout undo` / Helm rollback).
8. **Traceability:** tag every image with the Git commit hash; record build info so you can answer "what exactly is running in prod?"

**Sample pipeline structure:**
```
Build → Unit test → SonarQube → Docker build+push → Deploy DEV (auto)
     → Deploy STAGING (auto on tag) → integration tests → Approve (manual)
     → Deploy PROD (blue-green) → smoke test
```

---

## Q3. Jenkins Declarative vs Scripted pipeline?

- **Declarative:** simpler, structured (`pipeline { stages { stage { steps { } } } }`). Easy to read, good for most teams. Has `input` (approval) built in.
- **Scripted:** Groovy code, full flexibility, complex logic. Powerful but harder to maintain.
- **Answer:** "I prefer declarative for readability and built-in features like `input`, `post`, and parallel stages; I use scripted only when I need custom Groovy logic."

---

## Q4. What are the important stages in your Jenkins pipeline?

1. **Checkout** — pull code from Git (GitHub).
2. **Build** — Maven `mvn clean package` (compile + package JAR).
3. **Test** — run unit tests (`mvn test`), publish test reports.
4. **Static analysis** — SonarQube scan, enforce **quality gate** (fail build if coverage/bugs below threshold).
5. **Docker build** — `docker build` the image.
6. **Push** — `docker push` to Amazon ECR (tag = version + commit hash).
7. **Deploy** — `kubectl set image` / `kubectl apply` to EKS (or Helm install).
8. **Verify/Smoke** — check pod health, rollout status, call the app URL.
9. **Notify** — Slack/email on success/failure.

- `post { failure { ... } }` handles cleanup and notifications.

---

## Q5. How do you handle secrets in Jenkins?

- Jenkins **Credentials plugin** (username/password, SSH keys, secret text, secret file) — referenced by ID in the pipeline, never printed.
- `withCredentials([string(credentialsId: 'prod-db-pass', variable: 'DB_PASS')]) { ... }`
- For ECR: store AWS access keys as credentials, or better use an IAM role for the Jenkins server.
- **Rule:** never put passwords in the Jenkinsfile or in plain text logs — mask them.

---

## Q6. What is GitOps? Why is it popular for Kubernetes?

- **GitOps** = Git is the single source of truth. The desired state of the cluster lives in Git (YAML files).
- Tools: **Argo CD**, **Flux** — they watch Git and keep the cluster in sync. If someone changes the cluster manually, Argo CD reverts it.
- **Benefits:** rollback = `git revert`; audit = git history; review = PR process; no one SSHes into the cluster to change things.
- Good to mention: "We use GitOps principles — all our EKS manifests live in Git and deployments are reviewed as PRs."

---

## Q7. Git branching strategy — which one do you use?

- **GitFlow:** `main` (production), `develop`, feature branches, release branches. Good for release-based projects.
- **Trunk-based:** everyone commits to `main`/short-lived branches, small frequent merges — great for CI/CD (deploy on every merge).
- **For CI/CD-heavy teams:** trunk-based or a light GitFlow — short branches, PR review, merge → CI triggers deploy to dev.

---

## Q8. Blue-Green, Canary, Rolling — explain each simply.

- **Rolling:** update pods a few at a time. Old and new run together briefly. Default, zero downtime.
- **Blue-Green:** run BOTH versions fully (blue = old, green = new). When green is verified, switch the router/Service to green. **Rollback = switch back.** Needs double resources.
- **Canary:** send a small % of traffic to the new version (e.g., 5%), watch errors/latency, then ramp to 100%. Safest for production. Needs traffic routing (Istio/service mesh, or Ingress weights).

---

## Q9. How do you verify a deployment is successful after deploying?

**Post-deployment checks (matches your resume):**
1. `kubectl rollout status deployment/myapp` — rollout completed?
2. `kubectl get pods` — all Running, 0 restarts?
3. `kubectl get endpoints` — Service has endpoints (pods registered)?
4. Check logs — no errors: `kubectl logs deployment/myapp --tail=50`.
5. Health check / smoke test — call the app's health URL, verify HTTP 200.
6. Check metrics (CPU, error rate, latency) in Prometheus/Grafana or CloudWatch.
7. If any check fails → rollback.

---

## Quick commands to remember

| Concept | One-liner |
|---|---|
| CI | Build + test on every commit |
| CD | Deploy automatically (or with approval) |
| Build once, deploy many | Same image tag promoted across envs |
| Prod gate | `input 'Approve to deploy to prod?'` |
| Rollback | `kubectl rollout undo deployment/<name>` / Helm rollback |
| GitOps tools | Argo CD, Flux |
| Blue-green | Two full versions, switch router |
| Canary | 5% traffic → watch → 100% |