# Monitoring & Observability — Interview Q&A

**JD mentions CloudWatch, Prometheus, Grafana, Dynatrace.**

---

## Q1. What are the "Golden Signals"?

1. **Latency** — how long to serve a request.
2. **Traffic** — how much demand (requests/sec).
3. **Errors** — how many requests fail (rate, %).
4. **Saturation** — how "full" the service is (CPU, memory).

These tell you if the app is healthy from the USER's point of view.

---

## Q2. Prometheus + Grafana in Kubernetes — how does it work?

- **Prometheus:** scrapes metrics from pods/nodes (via exporters or `/metrics` endpoints), stores time-series data, evaluates alert rules (Alertmanager for notifications).
- **Grafana:** dashboards on top of Prometheus (visual graphs).
- In EKS you typically install the **kube-prometheus-stack** (Helm) — includes Prometheus, Grafana, Alertmanager, node exporters.
- **What I monitor:** CPU/memory per pod, pod restarts, 5xx error rate, response latency, disk space, HPA behavior.

---

## Q3. CloudWatch — what can it monitor?

- **Metrics:** CPU, memory, disk, network for EC2; custom metrics from your apps.
- **Logs:** CloudWatch Logs — collect app logs, search, metric filters, trigger alarms.
- **Alarms:** e.g., "CPU > 80% for 5 min" → SNS → email/PagerDuty.
- **Dashboards:** visual view of metrics.
- **CloudWatch vs Prometheus/Grafana:** CloudWatch is AWS-native (easy). Prometheus + Grafana are better for Kubernetes (pod-level metrics, richer queries). In EKS you often run both.

---

## Q4. How do you set up alerting / handle on-call?

- Define thresholds on key metrics (e.g., 5xx > 1% for 5 min, node memory > 85%).
- Alert → Alertmanager/CloudWatch alarm → SNS/Slack/PagerDuty → on-call engineer.
- **Alert quality:** every alert should be actionable, have a runbook, and not fire when nobody can fix it. Avoid alert fatigue.

---

## Q5. What is the difference between monitoring and observability?

- **Monitoring:** do we have a problem? (collecting metrics, logs, alerts on known things).
- **Observability:** WHY do we have a problem? (exploring unknown issues using logs, metrics, and traces together).
- **The 3 pillars:** Metrics (numbers), Logs (events/messages), Traces (request flow across services).
- Example: metric says 5xx errors rising → log shows a DB connection error → trace shows the failing service call.

---

## Quick commands / terms to remember

| Term | One-liner |
|---|---|
| Golden signals | Latency, Traffic, Errors, Saturation |
| Prometheus | Collects metrics + alerts |
| Grafana | Visualizes dashboards |
| Alertmanager | Sends alerts (Slack/PagerDuty) |
| CloudWatch | AWS metrics + logs + alarms |
| 3 pillars | Metrics, Logs, Traces |
| Dynatrace | Full-stack APM/observability tool |