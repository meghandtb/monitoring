Section 1: Conceptual Understanding (20 pts)
1. Explain the key differences between proactive and reactive monitoring. (5 pts)

The difference between proactive and reactive monitoring is related to what stage the issues are identitied and fixed.

Proctive monitoring implies continuously observing the systems and analyzing metrics in order to identify and fix incidents before they reach and impact the end users. It helps in early detection of anomalies and failures, reduces donwtime, improves performance and supports capacity planning. In this case, monitoring tools such as Prometheus and Datadog can come in handy; for example they can alert the engineers when memory usage keeps increasing until a certain threshold, so action can be taken before a failure occurs.

On the other hand, reactive monitoring means indentifying and fixing the issue after it already happened and impacted the end users. This can lead to downtime while the issue is being handled by the corresponding team. An example in this case can be a helpdesk ticket raised in tools such as ITSM after the user already spotted an issue.

3. What are MTTD and MTTR? Why are they important? (5 pts)
MTTD --> Mean Time to Detect  --> the average time taken to become aware of an issue after it occurs. A slow MTTD time can impact reliability, the system's uptime and even customer experience, while a faster time can reduce the impact of the damage.
MTTR --> mean time to resolve --> the average time taken to completely resolve an incident after it was detected.

Both MTTD and MTTR are key metrics for meeting SLAs and improving user experience and trust in a system. 

5. Describe a typical incident lifecycle and the role of DevOps in each stage. (5 pts)
The incident lifecycle management involves the following steps:

-  Detection: the monitoring tools such as Prometheus, Datadog (or the solutions implemented by the DevOps team) are used to identify an issue/an anomaly. The monitoring system detects an issue and sends an alert (through alerting systems such as Pager Duty) to the responsible team. DevOps ensures automated detection and alerting are up.
-  Triage & classification: the incident's priority and impact are assesse. DevOps helps in defining the severity/priority levels and escalation policies and making sure they are automated.
-  Response: the on-call team is notified of the incident and needs to act accordingly in order to mitigate the impact. The DevOps team needs to define auto-scaling, automates failover mechanisms in order to reduce the manual intervation in case of failure.
-  Resolution: the root cause of the incident is verified/discovered/fixed. In this phase, the DevOps engineers may need to apply patches, rollback strategies or reconfigure infrastructure.
-  Postincident/postmortem: review and document the lesson learned; The DevOps team needs to keep track of the incident and promote continuous improvement by updating playbooks, runbooks and monitoring configurations.

7. List three external monitoring platforms and their advantages. (5 pts)

External monitoring platforms are known for easy installation, usage and configuration. Three examples can be:

- Datadog - a cloud native, SaaS based monitoring system, which seamlessly integrates with tools such as Kubernetes, AWS, Docker.
- Dynatrace - also a SaaS solution which offers AI-powered observability with automatic root cause analysis; it helps in reducing manual troubleshooting.
- New Relic - focused on monitoring application performance by offering detailed traicing of the transactions and easy an easy debugging framewoek for microservices and APIs.

Section 2: Monitoring Tools Exploration (20 pts)
5. What is Prometheus used for in monitoring? (5 pts)
Prometheus is an open-source, self hosted monitoring solution, designed to scrape and collect data from its targets; it integrates with tools such as cAdvisor and NodeExporter in order to collect system metrics and send them to dashboarding solutions such as Grafana. Grafana helps with visualization and includes and Alertmanager component to notify teams when thresholds are reached.

6. Describe how Grafana complements Prometheus. (5 pts)
Grafana is a dashboard solution. It integrates with Prometheus and it provides a visual interface for the metrics that Prometheus collects, by turning it into customizable visualiations with graphs, charts and alerts. 
   
8. What data does Node Exporter collect? Name three example metrics. (5 pts)
Node exporter collects hardwared and OS level metrics from Linux/Unix hosts. These metrics usually reflect the helth of the infrastructure. Such metrics are:
- CPU usage and load averages
- Memory utilization
- Disk I/O and filesystem space usage
- Network traffic statistics
   
10. What is PagerDuty and how does it integrate with monitoring tools? (5 pts)
Pagerduty is an incident management and on-call schedulling platform, which intregates with monioring tools such as Prometheus, Grafana, Datadog, and performs the following:

- It allows the DevOps team to configure an on-call scheduling and alert routing as well as escalation policies. Escalation policies can also be set for unaknowledged alerts -> who to page when alerts remain ignored for a longer amount of time.
- It offers postmortems and runback integration.
- Reduces MTTR (Mean Time To Respond) by paging the right person.
- Automates incident response by integrating with collaboration tools such as Slack and Microsoft Teams.


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
   ```
   or in k9s <img width="1271" height="937" alt="image" src="https://github.com/user-attachments/assets/9d5508b9-3f58-4bbe-a930-b32fda00379c" />

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


## 🔹 Install Prometheus & Grafana __BONUS SOLUTION__

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

And now you have a functional monitoring stack.

## 🔹 6. Create Grafana Dashboard

Navigate to Dashboards -> New -> New Dashboard -> Add visualization 

<img width="2560" height="808" alt="image" src="https://github.com/user-attachments/assets/d8733e1e-14f6-4469-8a6c-fe9a92df12ec" />

Select Prometheus as the data source: 

<img width="1220" height="455" alt="image" src="https://github.com/user-attachments/assets/40bef5d4-276f-43ff-a2e0-6eeda3107bf1" />

### Panel 1 — CPU Usage per pod

```promql
sum(rate(container_cpu_usage_seconds_total{image!="", pod!=""}[5m])) by (pod, namespace)
```
<img width="868" height="699" alt="image" src="https://github.com/user-attachments/assets/fc4a4d26-0f32-4306-a820-82d0c0f3a04a" />


### Panel 2 — CPU Usage per node

```promql
sum(rate(container_cpu_usage_seconds_total{image!=""}[5m])) by (node)

```

<img width="869" height="669" alt="image" src="https://github.com/user-attachments/assets/08a10682-ef17-4b2c-bd0f-6397d5203928" />

### Panel 3 — Memory per pod (MB)

```promql
sum(container_memory_working_set_bytes{image!="", pod!=""}) by (pod, namespace) / 1024 / 1024
```
<img width="1210" height="719" alt="image" src="https://github.com/user-attachments/assets/b6edde39-b668-4897-ae27-d2a721dcb6ad" />

### Panel 4 — Memory per node (MB)

```promql
sum(container_memory_working_set_bytes{image!=""}) by (node) / 1024 / 1024
```
<img width="1201" height="740" alt="image" src="https://github.com/user-attachments/assets/1efab8cf-45ec-4cb1-af9e-f160e513b2f9" />

### Panel 5 — Container Resource Usage Overview

The panel showing container stats has a more complex setup, consisting of the following steps:

Select the *Table* visualization type (instead of *Time series* as the other ones).
This visualization is composed of 5 queries which reveal data about containers, such as memory, CPU, restarts. But first of all the variables used for these queries need to be defined. In the Dasboard Settings -> Edit -> Variables

<img width="1210" height="477" alt="image" src="https://github.com/user-attachments/assets/d434d89b-4a64-43b2-bb3d-3c89b255da90" />

```Classic Query -> namespace
label_values(kube_pod_info, namespace)
```

```Classic Query -> pod
label_values(kube_pod_info{namespace=~"$namespace"}, pod)
```

```Classic Query -> container
label_values(kube_pod_container_info{namespace=~"$namespace", pod=~"$pod"}, container)
```

This will help in grouping collected data for better readability.
For each variable maintain the same structure as bellow:

<img width="767" height="828" alt="image" src="https://github.com/user-attachments/assets/1135abd9-cc30-46f7-957a-d649983ec71a" />

After I configured these variables I can use them in the Dashboard to select for which namespace, pod and container I want the data to be shown.

<img width="794" height="131" alt="image" src="https://github.com/user-attachments/assets/f9fce383-6bce-4774-9f71-757c7cad4a3c" />


```promql
sum(rate(container_cpu_usage_seconds_total{
  image!="", 
  container!="POD", 
  container!="",
  namespace=~"$namespace",
  pod=~"$pod",
  container=~"$container"
}[5m])) by (namespace, pod, container)
```
<img width="870" height="684" alt="image" src="https://github.com/user-attachments/assets/c7925953-4a5c-4310-9005-037520da955b" />

In the *Options* section --> Legend --> Custom --> Give a suggestive name because this will be the a table column (in this particular case the value will be "CPU Usage" --> Format = Table --> Type = Instant (as seen in the image above)

The following queries will follow the same pattern. 

```promql
sum(container_memory_working_set_bytes{
  image!="", 
  container!="POD", 
  container!="",
  namespace=~"$namespace",
  pod=~"$pod",
  container=~"$container"
}) by (namespace, pod, container) / 1024 / 1024
```

<img width="870" height="699" alt="image" src="https://github.com/user-attachments/assets/e4faebd8-d811-49c1-9309-85f668b9bc04" />

```promql
sum(kube_pod_container_resource_limits{
  resource="memory",
  container!="",
  namespace=~"$namespace",
  pod=~"$pod",
  container=~"$container"
}) by (namespace, pod, container) / 1024 / 1024
```

<img width="871" height="561" alt="image" src="https://github.com/user-attachments/assets/bb36e526-7422-44d0-af86-de02edfe9c97" />

```promql
sum(kube_pod_container_resource_limits{
  resource="cpu",
  container!="",
  namespace=~"$namespace",
  pod=~"$pod",
  container=~"$container"
}) by (namespace, pod, container)
```

<img width="863" height="285" alt="image" src="https://github.com/user-attachments/assets/ea7e6f77-83c4-44c8-9bbb-a73a16c172e0" />

```promql
sum(kube_pod_container_status_restarts_total{
  namespace=~"$namespace",
  pod=~"$pod",
  container=~"$container"
}) by (namespace, pod, container)
```
<img width="850" height="252" alt="image" src="https://github.com/user-attachments/assets/60614177-a995-4448-a29c-57a5be24c613" />

In this stage the data collected does not make any sense, so we need to trasnform it and make it human readble.
In the *Transformations* tab select the following:
<img width="885" height="306" alt="image" src="https://github.com/user-attachments/assets/207803d8-201b-4994-8446-c06a347f0fd3" />

The *Merge series/tables* transformation does not need any further configuration, so select the following in the *Group by* one.
<img width="870" height="299" alt="image" src="https://github.com/user-attachments/assets/3626e545-657f-4834-85b3-c91f28f2b61c" />

This will help in presenting the data group and eliminating duplicates.

In the *Organize fields by name* select the following:
<img width="873" height="381" alt="image" src="https://github.com/user-attachments/assets/934a98e9-1db5-4e1b-bc09-f3f344e5322d" />

These values will be the column names so I needed them to be as easy to understand as possible.

<img width="1204" height="739" alt="image" src="https://github.com/user-attachments/assets/4f61d1f1-e791-4464-9706-2f9f12bac0e8" />

---

## 🔹 7. Alerts in Grafana (10 pts)

Access the newly created dashboard -->> CPU Usage per Node -->> Edit -->> Alert -->> New Alert Rule
Condition: 

<img width="1172" height="590" alt="image" src="https://github.com/user-attachments/assets/952ae348-8179-432c-bdc3-d2dabae7edf3" />

Configure a folder for the alerts and a duration.

<img width="830" height="813" alt="image" src="https://github.com/user-attachments/assets/fc61e838-f940-480a-be9e-b3582c080983" />

Add a contact point and configure a notification message.
<img width="794" height="803" alt="image" src="https://github.com/user-attachments/assets/8c01fea3-3137-4bfb-bd30-937b76a6798e" />

Save alert.

<img width="884" height="283" alt="image" src="https://github.com/user-attachments/assets/f45ebe54-6804-42a7-bf26-34dcc060a4d0" />


### Test the alert

Run a stress container:

```bash
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh
#and inside the container
while true; do :; done
```

Within 1–2 minutes, the Grafana alert should fire.

<img width="1935" height="828" alt="image" src="https://github.com/user-attachments/assets/2d3b20c5-00d8-46a8-bc32-28b7406d80d4" />

<img width="1921" height="896" alt="image" src="https://github.com/user-attachments/assets/bd96d44a-842c-44d1-b8fb-270fdde3921d" />

<img width="1918" height="907" alt="image" src="https://github.com/user-attachments/assets/0240a857-6b2b-4720-b6e9-1b014a3cfb34" />







