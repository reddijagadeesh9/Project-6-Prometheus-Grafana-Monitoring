
# Project 6: Monitoring and Alerting with Prometheus and Grafana

# 📌 Project Description

This project demonstrates the implementation of a complete Kubernetes observability and monitoring stack on AWS EKS using Prometheus, Grafana, Alertmanager, and kube-prometheus-stack.

The goal of this project is to monitor Kubernetes infrastructure, visualize metrics, configure alerting, and integrate Slack notifications for real-time incident monitoring.

This setup simulates a production-grade monitoring solution used in modern DevOps and SRE environments.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/572046b3-3993-4ad1-915a-255c68fc1b61" />

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
<img width="1557" height="340" alt="Screenshot 2026-05-07 171631" src="https://github.com/user-attachments/assets/bc49ca78-3865-4675-a577-30375002df64" />

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
<img width="1046" height="223" alt="Screenshot 2026-05-07 173054" src="https://github.com/user-attachments/assets/cfbd2200-0637-447c-ae9e-4b6cbd05def5" />

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
<img width="1557" height="340" alt="Screenshot 2026-05-07 171631" src="https://github.com/user-attachments/assets/f8875a81-1eba-4c84-a82f-efca720952ea" />

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
<img width="1068" height="241" alt="Screenshot 2026-05-07 201106" src="https://github.com/user-attachments/assets/79adba91-50ba-42c1-ac78-6e65843b7d68" />

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
<img width="386" height="587" alt="Screenshot 2026-05-07 181658" src="https://github.com/user-attachments/assets/88a129d0-faf4-4ea7-a118-8f228d285232" />

---

## Prometheus

```bash id="sy5xqn"
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090 --address 0.0.0.0
```

Access:

```text id="sz6xqo"
http://EC2_PUBLIC_IP:9090
```
<img width="532" height="585" alt="Screenshot 2026-05-07 181705" src="https://github.com/user-attachments/assets/a598087b-2669-4c87-b31b-cdd5ec9d8162" />

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

<img width="855" height="709" alt="Screenshot 2026-05-07 174013" src="https://github.com/user-attachments/assets/f43cacff-357f-4a06-aa79-31a4aae70e52" />

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
<img width="836" height="144" alt="Screenshot 2026-05-07 175516" src="https://github.com/user-attachments/assets/b22d101f-6bdb-40c5-a21f-45d298f5448d" />

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
---
Step 17 : Deploy Sample Monitoring Application

Created a sample Kubernetes application for monitoring demonstration.

deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: nginx
        ports:
        - containerPort: 80
---
---
Apply:

kubectl apply -f deployment.yaml

Output:

deployment.apps/demo-app created
---
---
#Step 18 : Expose Application Service
 
service.yaml

apiVersion: v1
kind: Service
metadata:
  name: demo-app-service
  labels:
    app: demo-app
spec:
  selector:
    app: demo-app
  ports:
  - port: 80
    targetPort: 80
---
Apply:
---
kubectl apply -f service.yaml
---
Output:
---
service/demo-app-service created
---
---
📌 Step 19 : Configure ServiceMonitor

Created ServiceMonitor resource so Prometheus can scrape application metrics automatically.

servicemonitor.yaml

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: demo-app-monitor
  labels:
    release: monitoring
spec:
  selector:
    matchLabels:
      app: demo-app
  endpoints:
  - port: http
    interval: 15s
---
Apply:
---
kubectl apply -f servicemonitor.yaml
---
Output:
---
servicemonitor.monitoring.coreos.com/demo-app-monitor created
---
<img width="748" height="262" alt="Screenshot 2026-05-07 174622" src="https://github.com/user-attachments/assets/ce9aeec6-d8f2-486a-8327-d9ca8ccc79e5" />

---
📌 Step 20: Verify Monitoring Pipeline

Verified:

Prometheus targets discovered
Metrics collection successful
Grafana dashboards displaying metrics
Alerts firing correctly
Slack notifications received
---
📌 Monitoring Components Installed
Component	Purpose
Prometheus	Metrics collection
Grafana	Dashboard visualization
Alertmanager	Alert routing
node-exporter	Node metrics
kube-state-metrics	Kubernetes object metrics
ServiceMonitor	Automatic service discovery
---
---
# 📌 Step 21: Configure ServiceMonitor

Created ServiceMonitor resource for Prometheus service discovery.
---

## servicemonitor.yaml
---
Configured:

* Automatic metrics scraping
* Application monitoring

---


# 📌 Step 22: Verify Slack Alerts

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

<img width="1046" height="223" alt="Screenshot 2026-05-07 173054" src="https://github.com/user-attachments/assets/6b548abd-b240-43e7-9e51-7565946337b4" />

---

## Monitoring Pods

<img width="1068" height="241" alt="Screenshot 2026-05-07 201106" src="https://github.com/user-attachments/assets/87131998-c344-49ee-8255-341967c86bdc" />


---

## Prometheus Targets



---

## Prometheus Alerts

<img width="1900" height="860" alt="Screenshot 2026-05-07 180133" src="https://github.com/user-attachments/assets/3b0dc9a4-1d66-4858-9239-008a119435f3" />

<img width="1906" height="871" alt="Screenshot 2026-05-07 180902" src="https://github.com/user-attachments/assets/802bd2f6-8cd8-45e7-806a-606c4ece3900" />

<img width="1897" height="863" alt="Screenshot 2026-05-07 185329" src="https://github.com/user-attachments/assets/9821acfe-57cd-4bfb-bf34-c4fa80bb4adc" />

<img width="1918" height="873" alt="Screenshot 2026-05-07 192833" src="https://github.com/user-attachments/assets/c4fb05ce-6e1d-477f-bc96-a2aec59f810e" />

<img width="1908" height="875" alt="Screenshot 2026-05-07 201748" src="https://github.com/user-attachments/assets/7150e96a-9ca1-474a-a012-1eb431fcac15" />

<img width="1883" height="416" alt="Screenshot 2026-05-07 201844" src="https://github.com/user-attachments/assets/e5f5974e-a06d-4e65-b167-01ad30e9dea6" />

---

## Grafana Dashboard

<img width="855" height="709" alt="Screenshot 2026-05-07 174013" src="https://github.com/user-attachments/assets/9c6b4720-fcb6-4955-bd2c-5821daa597ba" />

<img width="1892" height="853" alt="Screenshot 2026-05-07 202353" src="https://github.com/user-attachments/assets/2168c3a6-b3d4-40a2-b286-da83ed059759" />

<img width="1915" height="861" alt="Screenshot 2026-05-07 174948" src="https://github.com/user-attachments/assets/054e65b0-c848-4d38-aae1-64f97efa4109" />

<img width="1909" height="879" alt="Screenshot 2026-05-07 175003" src="https://github.com/user-attachments/assets/10014bc3-c906-45e1-b0f0-9dcd6ceda9e4" />

<img width="1901" height="856" alt="Screenshot 2026-05-07 175122" src="https://github.com/user-attachments/assets/0ec3b0f8-c414-4163-8bb4-b7a420020050" />

<img width="1918" height="870" alt="Screenshot 2026-05-07 175133" src="https://github.com/user-attachments/assets/7637adf8-a07f-49da-8f9b-59d79601d642" />

<img width="1917" height="860" alt="Screenshot 2026-05-07 175320" src="https://github.com/user-attachments/assets/cafa5c26-9ba0-4b49-8315-420dfcd4c5a3" />



---

## Slack Alerts

<img width="1912" height="798" alt="Screenshot 2026-05-07 192746" src="https://github.com/user-attachments/assets/ca70348f-0aa6-4a9f-ac2e-c53d8722ba32" />

<img width="1898" height="862" alt="Screenshot 2026-05-07 182354" src="https://github.com/user-attachments/assets/0cfabe18-aae3-4a6b-9a18-bfb5c3aa227e" />

<img width="936" height="767" alt="Screenshot 2026-05-07 192758" src="https://github.com/user-attachments/assets/29dc1674-fe63-4a20-8d1b-bcf5d9d75b15" />

---

# 📌 Repository Structure

```
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
