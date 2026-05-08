````
# Project 6: Monitoring and Alerting with Prometheus and Grafana

# 📌 Project Description

This project demonstrates the implementation of a complete Kubernetes observability and monitoring stack on AWS EKS using Prometheus, Grafana, Alertmanager, and kube-prometheus-stack.

The goal of this project is to monitor Kubernetes infrastructure, visualize metrics, configure alerting, and integrate Slack notifications for real-time incident monitoring.

This setup simulates a production-grade monitoring solution used in modern DevOps and SRE environments.

---

# 🚀 Technologies Used

- AWS EKS
- Kubernetes
- Prometheus
- Grafana
- Alertmanager
- Helm
- kube-prometheus-stack
- ServiceMonitor
- Slack Webhooks
- node-exporter
- kube-state-metrics
- kubectl
- eksctl

---

# 📌 Project Architecture

```text
                        +----------------------+
                        |      Slack Alerts    |
                        |  (#alerts Channel)   |
                        +----------+-----------+
                                   ^
                                   |
                          Alertmanager
                                   ^
                                   |
+-------------------------------------------------------------+
|                    Kubernetes Cluster (EKS)                 |
|                                                             |
|  +----------------+     +----------------+                  |
|  |   Prometheus   | --> |    Grafana     |                  |
|  | Metrics Engine |     | Dashboards UI  |                  |
|  +----------------+     +----------------+                  |
|           ^                                              |
|           |                                              |
|  +-------------------+   +--------------------+          |
|  | node-exporter     |   | kube-state-metrics |          |
|  +-------------------+   +--------------------+          |
|                                                             |
|  +------------------------------------------------------+  |
|  | Sample Application + ServiceMonitor + Alert Rules    |  |
|  +------------------------------------------------------+  |
|                                                             |
+-------------------------------------------------------------+
````

---

# 📌 Project Objectives

* Monitor Kubernetes cluster health
* Collect metrics from nodes and workloads
* Visualize infrastructure metrics using Grafana
* Configure Prometheus alerts
* Integrate Slack notifications using Alertmanager
* Monitor Kubernetes resources in real time

---

# 📌 Prerequisites

Before starting this project, install:

* AWS CLI
* kubectl
* eksctl
* Helm
* AWS IAM credentials configured

---

# 📌 Step 1: Launch EC2 Instance

Created Ubuntu EC2 instance for Kubernetes cluster management.

## Installed Required Tools

### Install AWS CLI

```bash id="sl1xqa"
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

### Install kubectl

```bash id="sm2xqb"
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

---

### Install eksctl

```bash id="sn3xqc"
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

---

### Install Helm

```bash id="so4xqd"
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

# 📌 Step 2: Configure AWS Credentials

```bash id="sp5xqe"
aws configure
```

Provided:

* AWS Access Key
* Secret Key
* Region
* Output format

---

# 📌 Step 3: Create EKS Cluster

Created Kubernetes cluster using eksctl.

```bash id="sq6xqf"
eksctl create cluster \
--name monitoring-cluster \
--region eu-north-1 \
--nodegroup-name monitoring-nodes \
--node-type m7i-flex.large \
--nodes 2 \
--managed
```

---

# 📌 Step 4: Verify Cluster

```bash id="sr7xqg"
kubectl get nodes
```

Verified:

* Worker nodes
* Cluster connectivity
* Node status Ready

---

# 📌 Step 5: Create Monitoring Namespace

```bash id="ss8xqh"
kubectl create namespace monitoring
```

---

# 📌 Step 6: Add Helm Repository

```bash id="st9xqi"
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update
```

---

# 📌 Step 7: Install kube-prometheus-stack

Installed:

* Prometheus
* Grafana
* Alertmanager
* node-exporter
* kube-state-metrics

```bash id="su1xqj"
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring
```

---

# 📌 Step 8: Verify Monitoring Stack

```bash id="sv2xqk"
kubectl get pods -n monitoring
```

Verified:

* Prometheus running
* Grafana running
* Alertmanager running
* Exporters running

---

# 📌 Step 9: Configure Port Forwarding

## Grafana

```bash id="sw3xql"
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80 --address 0.0.0.0
```

Access:

```text id="sx4xqm"
http://EC2_PUBLIC_IP:3000
```

---

## Prometheus

```bash id="sy5xqn"
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090 --address 0.0.0.0
```

Access:

```text id="sz6xqo"
http://EC2_PUBLIC_IP:9090
```

---

# 📌 Step 10: Login to Grafana

## Get Grafana Password

```bash id="ta7xqp"
kubectl get secret -n monitoring monitoring-grafana \
-o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

Credentials:

* Username: admin
* Password: Retrieved from secret

---

# 📌 Step 11: Configure Prometheus Alert Rules

Created custom alert rules.

## alerts.yaml

```yaml id="tb8xqq"
groups:
- name: custom.rules
  rules:
  - alert: AlwaysFiringTest
    expr: vector(1)
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Test alert is firing"
```

Apply:

```bash id="tc9xqr"
kubectl apply -f alerts.yaml
```

---

# 📌 Step 12: Verify Prometheus Alerts

Open:

```text id="td1xqs"
http://EC2_PUBLIC_IP:9090/alerts
```

Verified:

* Alert rules loaded
* Alerts in FIRING state

---

# 📌 Step 13: Configure Slack Integration

Configured Alertmanager with Slack Incoming Webhook for real-time notifications.

## Created Slack Webhook Secret

```bash id="te2xqt"
kubectl create secret generic alertmanager-slack-secret \
-n monitoring \
--from-literal=slack_api_url='REPLACE_WITH_SLACK_WEBHOOK'
```

---

# 📌 Step 14: Configure Alertmanager

Created Alertmanager configuration for Slack routing.

## alertmanager.yaml

Configured:

* Slack receiver
* Alert routing
* Notification formatting

---

# 📌 Step 15: Create Alertmanager Secret

```bash id="tf3xqu"
gzip -c alertmanager.yaml > alertmanager.yaml.gz

kubectl create secret generic \
alertmanager-monitoring-kube-prometheus-alertmanager-generated \
-n monitoring \
--from-file=alertmanager.yaml.gz=alertmanager.yaml.gz
```

---

# 📌 Step 16: Restart Alertmanager

```bash id="tg4xqv"
kubectl delete pod -n monitoring \
-l app.kubernetes.io/name=alertmanager
```

Verified:

* Alertmanager restarted successfully
* Slack notifications working

---

# 📌 Step 17: Configure ServiceMonitor

Created ServiceMonitor resource for Prometheus service discovery.

## servicemonitor.yaml

Configured:

* Automatic metrics scraping
* Application monitoring

---

# 📌 Step 18: Deploy Sample Application

## deployment.yaml

Created Kubernetes deployment for testing monitoring stack.

Apply:

```bash id="th5xqw"
kubectl apply -f deployment.yaml
```

---

# 📌 Step 19: Expose Application Service

## service.yaml

Created Kubernetes service.

Apply:

```bash id="ti6xqx"
kubectl apply -f service.yaml
```

---

# 📌 Step 20: Verify Slack Alerts

Verified:

* Alertmanager notifications
* Slack alerts received successfully
* Real-time monitoring pipeline operational

---

# 📌 Monitoring Components Installed

| Component          | Purpose                    |
| ------------------ | -------------------------- |
| Prometheus         | Metrics collection         |
| Grafana            | Dashboards & visualization |
| Alertmanager       | Alert routing              |
| node-exporter      | Node metrics               |
| kube-state-metrics | Kubernetes metrics         |
| ServiceMonitor     | Metrics scraping           |

---

# 📌 Grafana Dashboards

Imported:

* Kubernetes Cluster Monitoring
* Node Exporter Full
* Kubernetes Pod Monitoring

Metrics monitored:

* CPU usage
* Memory usage
* Pod health
* Node status
* Resource utilization

---

# 📌 Screenshots

## EKS Cluster

![EKS Cluster](screenshots/eks-cluster.png)

---

## Monitoring Pods

![Monitoring Pods](screenshots/monitoring-pods.png)

---

## Prometheus Targets

![Prometheus Targets](screenshots/prometheus-targets.png)

---

## Prometheus Alerts

![Prometheus Alerts](screenshots/prometheus-alerts.png)

---

## Grafana Dashboard

![Grafana Dashboard](screenshots/grafana-dashboard.png)

---

## Slack Alerts

![Slack Alerts](screenshots/slack-alerts.png)

---

# 📌 Repository Structure

```text id="tj7xqy"
Project-6-Prometheus-Grafana-Monitoring/
│
├── helm/
│   └── values.yaml
│
├── manifests/
│   ├── alerts.yaml
│   ├── sample-app-deployment.yaml
│   ├── sample-app-service.yaml
│   ├── servicemonitor.yaml
│   └── alertmanager-config.yaml
│
├── screenshots/
│   ├── eks-cluster.png
│   ├── monitoring-pods.png
│   ├── prometheus-alerts.png
│   ├── grafana-dashboard.png
│   └── slack-alerts.png
│
├── README.md
└── .gitignore
```

---

# 📌 Challenges Faced

| Challenge                     | Solution                               |
| ----------------------------- | -------------------------------------- |
| Alertmanager config issues    | Used direct secret-based configuration |
| Slack webhook push protection | Removed secrets from Git history       |
| Port forwarding from EC2      | Configured `--address 0.0.0.0`         |
| PrometheusRule YAML issues    | Corrected rule formatting              |

---

# 📌 Best Practices Followed

* Namespace isolation for monitoring stack
* Helm-based Kubernetes package management
* Secret management for webhooks
* Reusable Kubernetes manifests
* ServiceMonitor for dynamic scraping
* Production-style monitoring setup

---

# 📌 Outcome

Successfully implemented a production-grade Kubernetes monitoring and observability stack capable of:

* Infrastructure monitoring
* Metrics collection
* Dashboard visualization
* Real-time Slack alerting
* Kubernetes health monitoring
* Production-style incident monitoring

---

# 📌 Future Enhancements

* Loki Log Aggregation
* PagerDuty Integration
* SLO Dashboards
* Persistent Metrics Storage
* Grafana Provisioning
* GitOps Deployment
* Long-term Metrics Retention

---

# 📌 Author

## REDDI JAGADEESWARA RAO

DevOps Engineer | AWS | Kubernetes | Terraform | CI/CD | Observability

---

```
```
