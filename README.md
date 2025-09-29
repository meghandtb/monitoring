Section 1: Conceptual Understanding (20 pts)
1. Explain the key differences between proactive and reactive monitoring. (5 pts)

The difference between proactive and reactive monitoring is related to what stage the issues are identitied and fixed.
Proctive monitoring implies continually observing the systems in order to identify and fix incidents before affecting the end users. It helps in early detection of anomalies and failures, reduces donwtime and improves performance.
On the other hand reactive monitoring means indentifying and fixing the issue after it already happened and impacted the end users. This can lead to downtime while the issue is being handled by the corresponding team.

3. What are MTTD and MTTR? Why are they important? (5 pts)
MTTD --> mean time to detect  --> the average time taken to detect an issue
MTTR --> mean time to resolve --> the average time of fixing the issues

5. Describe a typical incident lifecycle and the role of DevOps in each stage. (5 pts)
The incident lifecycle management involves the following steps:

-  Detection: the monitoring tools(Prometheus, Datadog)/solution implemented by the DevOps team are used to identify an issue/an anomaly. The monitoring system detects an issue and sends an alert (through alerting systems such as Pager Duty) to the responsible team.
-  Response: the team is notified of the incident and needs to act accordingly in order to mitigate the impact
-  Resolution: the root cause of the incident is verified/discovered/fixed
-  Postincident/postmortem: review and document the lesson learned

7. List three external monitoring platforms and their advantages. (5 pts)

Datadog is a cloud native, SaaS based monitoring system, which seamlesly integrates with tools such as K8s, AWS.
Dynatrace

Section 2: Monitoring Tools Exploration (20 pts)
5. What is Prometheus used for in monitoring? (5 pts)
Prometheus is a self hosted monitoring solution, used in scraping the data from the monitored resources; it integrates with tools such as cAdvisor, NodeExporter in order to colect system metrics and sends those metrics to dashboarding solutions such as Grafana, which grafically desplays the sourced data.

6. Describe how Grafana complements Prometheus. (5 pts)
Grafana is a dashboard solution, it integrates with Prometheus and uses it as a data source for the metrics that it will represent in a dashboard form; it servers as a visual interface for the users of the monitoring solution
   
8. What data does Node Exporter collect? Name three example metrics. (5 pts)
Node exporter collects hardwared and OS metrics -> system metrics such as CPU/memory/disk.
   
10. What is PagerDuty and how does it integrate with monitoring tools? (5 pts)
Pagerduty is an incident management platform which does the following:

- Integrates with monittoring tools such as Datadog/Prometheus
- It allows the DevOps team to configure an on-call scheduling and alert routing as well as escalation policies for unacknowledged alerts -> who to escalate based on priority
- It offers postmortems and runback integration
- Reduces MTTR (Mean Time To Respond) by paging the right person


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

   <img width="644" height="254" alt="image" src="https://github.com/user-attachments/assets/9f6bcc28-7e77-4877-bace-d00b98c4265d" />


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


__BONUS SOLUTION__

Install all of the requirements through the kube-prometheus helm chart. This includes the Prometheus, Grafana and Node Exporter packages and requires little to no configuration.
Install the chart with the following command:

```
helm install kube-prometheus  oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack
```

<img width="1172" height="420" alt="image" src="https://github.com/user-attachments/assets/aab66b0a-2b34-46bc-844b-d1a3f7b39e59" />

Get the password for the Grafana dashboard:

```
kubectl --namespace default get secrets kube-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```
Access the Grafana local instance:

```
export POD_NAME=$(kubectl --namespace default get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kube-prometheus" -oname)
kubectl --namespace default port-forward $POD_NAME 300
```

It is now available on port 3000 on the local host:
<img width="740" height="152" alt="image" src="https://github.com/user-attachments/assets/981a8348-7207-4578-800f-6de2af038644" />
<img width="1250" height="901" alt="image" src="https://github.com/user-attachments/assets/41a46307-14ca-4702-b091-9dd3a0abe2fd" />

Use the credentials generated by the previous command and access the dashboard.

<img width="2559" height="1299" alt="image" src="https://github.com/user-attachments/assets/d527ce16-06f7-4929-8bb5-21d4a3079765" />

Configure the dashboard to match the assigment requirements:


For creating an alert for CPU usage:

Access the newly created dashboard -->> CPU Usage Visualization -->> Edit -->> Alert -->> New Alert Rule

<img width="1923" height="770" alt="image" src="https://github.com/user-attachments/assets/d9869e99-db18-4eae-b0f9-dce2f00ef957" />

<img width="1935" height="828" alt="image" src="https://github.com/user-attachments/assets/2d3b20c5-00d8-46a8-bc32-28b7406d80d4" />

<img width="1921" height="896" alt="image" src="https://github.com/user-attachments/assets/bd96d44a-842c-44d1-b8fb-270fdde3921d" />

<img width="1918" height="907" alt="image" src="https://github.com/user-attachments/assets/0240a857-6b2b-4720-b6e9-1b014a3cfb34" />







