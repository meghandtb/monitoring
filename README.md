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

## 🔹 1. Environment Setup

1. Install prerequisites:

   ```bash
   docker
   kind
   k9s
   helm
   ```

2. Create a namespace and a kind cluster:

   ```bash
   kubectl create namespace metrice
   kind create cluster --config kind.yaml 
   ```

3. Verify:

   ```bash
   kubectl get nodes
   or in k9s <img width="1271" height="937" alt="image" src="https://github.com/user-attachments/assets/9d5508b9-3f58-4bbe-a930-b32fda00379c" />

   ```

---

## 🔹 2. Install Prometheus & Grafana

*Prometheus*

I used the prometheus-comunity helm chart.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Verify installation:

```bash
kubectl get pods -n metrics
```
<img width="1500" height="276" alt="image" src="https://github.com/user-attachments/assets/48e79c5e-e905-4445-9670-e8727a488956" />

---

```
helm install prometheus prometheus-community/prometheus -n metrics
```
Check that Prometheus is running on http://localhost:9090 after port forwarding

<img width="975" height="757" alt="image" src="https://github.com/user-attachments/assets/83c680d3-30ca-4bbb-b136-f86168b0a639" />

*Grafana*

```
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
helm install grafana grafana/grafana -n metrics
```

Check that Grafana pod is running.

<img width="1503" height="420" alt="image" src="https://github.com/user-attachments/assets/d9824c8d-ea3c-40c0-aee5-4333d6955a9a" />

Port forward the Grafana service and check that is running on https://localhost:3000.

```
kubectl port-forward svc/grafana 3000:80 -n metrics

```

<img width="1013" height="742" alt="image" src="https://github.com/user-attachments/assets/acde7a74-7156-4d9e-90a2-1a40b65399c2" />



## 🔹 3. Node Exporter Setup

Node Exporter is deployed automatically through the prometheus-community helm chart as a **DaemonSet**. Prometheus is pre-configured to scrape it.


<img width="1466" height="152" alt="image" src="https://github.com/user-attachments/assets/e4423e95-729e-4c09-8905-aea9ece26799" />

Node exporter is already a defined metric as a target for Prometheus.

<img width="1446" height="781" alt="image" src="https://github.com/user-attachments/assets/e3d858c3-9a5c-49bf-b1f8-9217dcc74fd8" />

---

## 🔹 4. cAdvisor Metrics

In Kubernetes, **kubelet already embeds cAdvisor** and exposes metrics at `/metrics/cadvisor`. The Helm chart configures Prometheus to scrape these automatically, so you don’t need to deploy a separate DaemonSet.

<img width="1506" height="653" alt="image" src="https://github.com/user-attachments/assets/cd2ddb90-c54c-4411-a016-9572301c42bd" />


## 🔹 5. Grafana Setup

Grafana is already accesible through port 3000 on localhost. The next step is to get the login credentials, where username is _admin_ and the password can be found out like so:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

* Username: `admin`
* Password: (output above)

Add the Prometheus data source under **Connections → Data sources**.

Since both Prometheus and Grafana are deployed inside the cluster, the data source can be specified as a fqdn.

<img width="1098" height="560" alt="image" src="https://github.com/user-attachments/assets/3933284a-001b-41f2-835e-c9d64aeaf14f" />

Test and save the connection.

<img width="808" height="248" alt="image" src="https://github.com/user-attachments/assets/0c684c52-ffaa-4544-8f01-46d93c115a4b" />


---

## 🔹 6. Create Grafana Dashboard

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


