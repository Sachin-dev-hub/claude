# Behavioral & Scenario Questions — Interview Q&A

---

## Q1. Tell me about a production incident you handled. (STAR method)

**Use STAR: Situation, Task, Action, Result.**
Example from your experience:

- **Situation:** After a release, application pods were CrashLoopBackOff in production.
- **Task:** Restore service quickly and find root cause.
- **Action:** Checked `kubectl get pods` → saw CrashLoopBackOff → `kubectl logs` → found a missing environment variable referenced in config → rolled back to previous image (`kubectl rollout undo`) to restore service, then fixed the ConfigMap and redeployed.
- **Result:** Service restored in ~10 minutes, incident documented, RCA shared, added a pre-deploy check so it doesn't happen again (post-deploy verification in the pipeline).

---

## Q2. How do you handle on-call / a page at 2 AM?

1. Acknowledge the alert immediately.
2. Assess severity — is it user-impacting?
3. Stabilize FIRST (rollback, scale up, restart) — fix permanently later.
4. Keep calm, communicate status to the team/lead.
5. Post-incident: root cause analysis (RCA), fix, add monitoring/alert, write a runbook.

---

## Q3. Why do you want this role / why Sr DevOps?

- Show the link: "I have 6.5+ years across AWS, Kubernetes, CI/CD. This role is about building platform capabilities for a data platform — exactly where I want to grow, especially EKS and Ansible automation at scale. I'm comfortable taking ownership of infrastructure decisions and mentoring juniors."

---

## Q4. How do you learn a new tool quickly?

- Read official docs + quickstart, spin it up in a sandbox, build a small proof-of-concept, follow best practices from docs/blogs, use Kubernetes Slack/Stack Overflow, then apply to real work with code review.

---

## Q5. How do you handle a disagreement with a developer/teammate?

- Listen first, understand their goal, explain the trade-offs with facts (not opinions), find a middle ground (e.g., "let's do it your way in dev and test it"), and if needed escalate with data. Focus on what's best for the product, not who's right.

---

## Q6. What does "DevSecOps" mean to you? (it's in your summary)

- Security is part of EVERY stage of the pipeline, not an afterthought:
  - **Code stage:** SonarQube static analysis.
  - **Build stage:** scan dependencies for vulnerabilities.
  - **Image stage:** scan container images (Trivy/ECR scanning).
  - **Deploy stage:** IAM least-privilege, secrets in Secrets Manager/Vault (not in code).
  - **Run stage:** RBAC on the cluster, network policies, audit logs.
- DevSecOps = developers, ops, and security work together from the start.

---

## Q7. Why did you move from Linux Admin to DevOps?

- "As a Linux admin I automated repetitive server tasks with scripts. That pushed me toward DevOps — automation, CI/CD, containers, and cloud. DevOps lets me build systems that scale instead of managing servers one by one."

---

## Q8. Where do you see yourself in 3 years?

- "Growing into a platform/DevOps lead — owning infrastructure architecture, cloud migration, and mentoring a small team, while staying hands-on with Kubernetes and cloud-native tools."

---

## Q9. How do you keep up with new DevOps technologies?

- Follow Kubernetes/AWS release notes, read DevOps blogs (InfoQ, DevOps.com), practice in a lab (personal AWS account / minikube), attend meetups, and try new tools in small side projects before recommending them at work.

---

## Quick STAR template to memorize

| Letter | Meaning | Example line |
|---|---|---|
| **S** | Situation | "We had an outage after a release..." |
| **T** | Task | "I needed to restore service fast and find the cause." |
| **A** | Action | "I checked pod status, found the error in logs, rolled back..." |
| **R** | Result | "Service was back in 10 minutes, and we added a check to prevent it." |