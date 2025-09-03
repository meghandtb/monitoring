Section 1: Conceptual Understanding (20 pts)
1. Explain the key differences between proactive and reactive monitoring. (5 pts)
2. What are MTTD and MTTR? Why are they important? (5 pts)
3. Describe a typical incident lifecycle and the role of DevOps in each stage. (5 pts)
4. List three external monitoring platforms and their advantages. (5 pts)

Section 2: Monitoring Tools Exploration (20 pts)
5. What is Prometheus used for in monitoring? (5 pts)

6. Describe how Grafana complements Prometheus. (5 pts)
   
7. What data does Node Exporter collect? Name three example metrics. (5 pts)
   
8. What is PagerDuty and how does it integrate with monitoring tools? (5 pts)

Section 3: Hands-On Project – Build a Self-Hosted Monitoring Stack (60 pts)

# 📊 Kubernetes Monitoring Stack on kind

This project sets up a **self-hosted monitoring stack** on a Kubernetes cluster running in [kind](https://kind.sigs.k8s.io/). The stack includes:

* **Prometheus** – metrics collection
* **Grafana** – visualization and alerting
* **Node Exporter** – node/system metrics
* **cAdvisor (via kubelet)** – container-level metrics

---

## 🔹 1. Environment Setup (5 pts)

1. Install prerequisites:

   ```bash
   docker --version
   kind --version
   kubectl version --client
   helm version
   ```

2. Create a kind cluster:

   ```bash
   kind create cluster --name monitoring-cluster
   ```

3. Verify:

   ```bash
   kubectl get nodes
   ```

---

## 🔹 2. Install Prometheus & Grafana (10 pts)

We use the **kube-prometheus-stack** Helm chart, which bundles Prometheus, Grafana, Node Exporter, and kube-state-metrics.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

Verify installation:

```bash
kubectl get pods -n monitoring
```

Expected: Prometheus, Grafana, Node Exporter, and supporting pods should be running.

---

## 🔹 3. Node Exporter (5 pts)

Node Exporter is deployed automatically by kube-prometheus-stack as a **DaemonSet**. Prometheus is pre-configured to scrape it.

Check:

```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=node-exporter
```

You should see one `node-exporter` pod per cluster node.

---

## 🔹 4. cAdvisor Metrics (5 pts)

In Kubernetes, **kubelet already embeds cAdvisor** and exposes metrics at `/metrics/cadvisor`. The Helm chart configures Prometheus to scrape these automatically, so you don’t need to deploy a separate DaemonSet.

Check Prometheus targets:

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090
```

Open [http://localhost:9090/targets](http://localhost:9090/targets) → look for `kubelet` targets with `/metrics/cadvisor`.

---

## 🔹 5. Grafana Setup (5 pts)

Expose Grafana locally:

```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Access: [http://localhost:3000](http://localhost:3000)

Get credentials:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

* Username: `admin`
* Password: (output above)

Verify Prometheus data source under **Connections → Data sources**.

---

## 🔹 6. Create Grafana Dashboard (10 pts)

### Panel 1 — CPU Usage

```promql
rate(container_cpu_usage_seconds_total{container!="",pod!=""}[5m])
```

Visualization: Time series

### Panel 2 — Memory Usage

```promql
container_memory_usage_bytes{container!="",pod!=""}
```

Visualization: Gauge or Time series (convert bytes → MB/GB)

### Panel 3 — Container Count

```promql
count(container_memory_usage_bytes{container!=""})
```

Visualization: Stat

Save dashboard as **System & Container Overview**.

📸 *Screenshot placeholder: Dashboard with CPU, Memory, and Container panels.*

---

## 🔹 7. Alerts in Grafana (10 pts)

1. Edit **CPU Usage panel** → **Alert → Create alert rule**.
2. Expression (CPU %):

   ```promql
   avg(rate(container_cpu_usage_seconds_total{container!="",pod!=""}[5m])) * 100
   ```
3. Condition: **IS ABOVE 80 for 5m**.
4. Save alert.

📸 *Screenshot placeholder: Alert rule configuration.*

### Test the alert

Run a stress container:

```bash
kubectl run cpu-stress --rm -it --image=alpine -- /bin/sh
apk add --no-cache stress
stress --cpu 1 --timeout 300
```

Within 1–2 minutes, the Grafana alert should fire.

📸 *Screenshot placeholder: Alert firing (red state).*

---

## 🔹 8. Documentation & Deliverables (5 pts)

This README documents:

* Setup of kind cluster
* Installation of Prometheus, Grafana, Node Exporter
* Validation of cAdvisor metrics
* Grafana dashboard creation
* Alert rule setup and testing

Include screenshots in your submission for verification.

---

## ✅ Grading Checklist

* [x] Virtual machine or container host setup (**5 pts**)
* [x] Install Prometheus & Grafana + scrape configs (**10 pts**)
* [x] Node Exporter configured (**5 pts**)
* [x] cAdvisor configured (via kubelet) (**5 pts**)
* [x] Grafana connected to Prometheus (**5 pts**)
* [x] Dashboard with CPU, Memory, Container stats (**10 pts**)
* [x] Alert rule configured and tested (**10 pts**)
* [x] Documentation with screenshots (**5 pts**)
