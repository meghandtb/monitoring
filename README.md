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

Section 3: Hands-On Project – Build a Self-Hosted Monitoring Stack (60
pts)
This is a full hands-on practical project. You are required to set up a self-hosted monitoring
stack using Prometheus, Grafana, Node Exporter, and cAdvisor, then configure Grafana to
display metrics and set up a functional alert.
9. Set up a virtual machine or container host for your monitoring stack. (5 pts)



10. Install Grafana and Prometheus and configure it with the correct scrape configs for Node
Exporter and cAdvisor. (10 pts)
11. Install and configure Node Exporter to expose system metrics. (5 pts)
12. Install and configure cAdvisor for container monitoring. (5 pts)
13. Install Grafana and connect it to Prometheus as a data source. (5 pts)
14. Create a dashboard in Grafana with panels showing CPU, Memory, and Container stats.
(10 pts)
15. Set up a threshold alert on Grafana (e.g., CPU usage > 80%) and test it. (10 pts)
16. Document your steps clearly in a README.md and include screenshots. (5 pts)
