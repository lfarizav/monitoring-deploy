<table style="border-collapse: collapse; border: none;">
  <tr style="border-collapse: collapse; border: none;">
    <td style="border-collapse: collapse; border: none;">
      <a href="http://www.openairinterface.org/">
         <img src="https://gitlab.eurecom.fr/uploads/-/system/user/avatar/716/avatar.png?width=800" alt="" border=3 height=50 width=50>
         </img>
      </a>
    </td>
    <td style="border-collapse: collapse; border: none; vertical-align: center;">
      <b><font size = "5">Tenant Repository for Monitoring Stack (Prometheus + Grafana)</font></b>
    </td>
  </tr>
</table>

# Author
**Luis Felipe Ariza Vesga** 
emails: lfarizav@gmail.com, lfarizav@unal.edu.co
> **Tenant Repository for Monitoring Stack (Prometheus + Grafana)**

This repository contains the Helm charts and Flux CD configurations for deploying a complete monitoring and observability stack on Kubernetes using the kube-prometheus-stack.

[![Flux](https://img.shields.io/badge/Flux-Managed-blue)](https://fluxcd.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Visualization-blue)](https://grafana.com/)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Deployment](#deployment)
- [Configuration](#configuration)
- [Access](#access)
- [Dashboards](#dashboards)
- [Alerting](#alerting)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

This tenant repository is managed by the **k8s-open5gs-fleet** GitOps fleet and deploys:

- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **Alertmanager**: Alert routing and management
- **Node Exporter**: Node-level metrics
- **Kube State Metrics**: Kubernetes object metrics
- **Prometheus Operator**: Kubernetes-native Prometheus management

### Key Features

- ✅ **GitOps deployment** via Flux CD
- ✅ **Complete observability** stack
- ✅ **Pre-configured dashboards** for Kubernetes
- ✅ **Service discovery** for automatic scraping
- ✅ **Alert rules** for common scenarios
- ✅ **Slack integration** for notifications
- ✅ **Persistent storage** for metrics retention
- ✅ **High availability** support

---

## 📁 Repository Structure

```
monitoring-deploy/
├── flux/
│   ├── base/                               # Base Flux configurations
│   │   ├── kustomization.yaml              # Aggregates all resources
│   │   ├── monitoring-gitrepository.yaml   # Git source definition
│   │   ├── monitoring-helmrelease.yaml     # Helm release config
│   │   ├── monitoring-alert.yaml           # GitHub webhook alert
│   │   └── slack-notif-alert.yaml          # Slack notifications
│   │
│   └── dev/                                # Development environment
│       └── kustomization.yaml              # Dev-specific overlays
│
├── helm/
│   └── charts/
│       └── kube-prometheus-stack/          # Main Helm chart
│           ├── Chart.yaml                  # Chart metadata
│           ├── Chart.lock                  # Dependencies lock
│           ├── values.yaml                 # Default values
│           ├── templates/                  # K8s manifests
│           │   ├── prometheus/             # Prometheus resources
│           │   ├── alertmanager/           # Alertmanager resources
│           │   └── grafana/                # Grafana resources
│           └── charts/                     # Sub-charts
│               ├── prometheus/
│               ├── grafana/
│               ├── alertmanager/
│               ├── kube-state-metrics/
│               └── prometheus-node-exporter/
│
├── README.md                               # This file
└── LICENSE
```

---

## 🏗️ Architecture

### Monitoring Stack Components

```
┌─────────────────────────────────────────────────────────────┐
│                      monitoring namespace                   │
│                                                             │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │  Prometheus  │◄────►│ Alertmanager │                    │
│  │   :9090      │      │   :9093      │                    │
│  └──────┬───────┘      └──────┬───────┘                    │
│         │                     │                             │
│         │                     │                             │
│         ▼                     ▼                             │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │   Grafana    │      │    Slack     │                    │
│  │   :3000      │      │ Notifications│                    │
│  └──────────────┘      └──────────────┘                    │
│         ▲                                                   │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │
          │ Scrapes Metrics
          │
┌─────────┴───────────────────────────────────────────────────┐
│                    Metric Sources                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Node Exporter│  │ Kube State   │  │  Application │     │
│  │              │  │   Metrics    │  │   Metrics    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Flux CD      │  │   Keptn      │  │  Open5GS     │     │
│  │  Metrics     │  │   Metrics    │  │   Metrics    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Metric Flow

```
Application/Service
       │
       │ Expose metrics endpoint
       ▼
ServiceMonitor (CRD)
       │
       │ Discovered by Prometheus Operator
       ▼
Prometheus Server
       │
       ├─► Evaluate Alert Rules
       │        │
       │        ├─► Alertmanager ─► Slack
       │        └─► PagerDuty/Email
       │
       └─► Store Time Series Data
                │
                └─► Query by Grafana ─► Dashboard
```

---

## 🔧 Prerequisites

### Fleet Repository Setup

This tenant is managed by the fleet repository. Ensure:

1. **Fleet repo** is bootstrapped: [k8s-open5gs-fleet](https://github.com/Cuemby/k8s-open5gs-fleet)
2. **Namespace** `monitoring` exists
3. **RBAC** is configured
4. **GitRepository** resource points to this repo

### Required Resources

- Kubernetes cluster (v1.27+)
- 4GB+ RAM for Prometheus
- Persistent storage (for metrics retention)
- Flux CD installed and running

### Storage Requirements

- **Prometheus**: 50Gi+ for production (configurable)
- **Grafana**: 10Gi for dashboards and data
- **Alertmanager**: 10Gi for alert history

---

## 🚀 Deployment

### Option 1: Initial Chart Setup (First Time)

If you're setting up this repository for the first time, download and customize the official kube-prometheus-stack Helm chart:

<details>
<summary><b>📦 Download Official Helm Chart</b></summary>

#### Step 1: Add Prometheus Community Helm Repository

```bash
# Add the official Prometheus Community Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

#### Step 2: Download the Chart

```bash
# Create helm/charts directory if it doesn't exist
mkdir -p helm/charts

# Pull the chart (replace X.Y.Z with desired version)
helm pull prometheus-community/kube-prometheus-stack \
  --version 56.6.2 \
  --untar \
  --untardir helm/charts
```

#### Step 3: Customize values.yaml

Edit `helm/charts/kube-prometheus-stack/values.yaml` to match your environment:

```yaml
# helm/charts/kube-prometheus-stack/values.yaml

# Prometheus Configuration
prometheus:
  enabled: true
  
  prometheusSpec:
    # Retention settings
    retention: 30d
    retentionSize: "45GB"
    
    # Persistent storage
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi
    
    # Resource limits
    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: 2000m
        memory: 4Gi
    
    # Service monitors selector (watch all namespaces)
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false
    
    # Enable additional scrape configs
    additionalScrapeConfigs: []

# Grafana Configuration
grafana:
  enabled: true
  
  # Admin credentials
  adminPassword: "admin"  # Change in production!
  
  # Persistent storage
  persistence:
    enabled: true
    size: 10Gi
  
  # Resources
  resources:
    requests:
      cpu: 100m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
  
  # Pre-installed dashboards
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
      - name: 'default'
        orgId: 1
        folder: ''
        type: file
        disableDeletion: false
        editable: true
        options:
          path: /var/lib/grafana/dashboards/default
  
  # Data sources
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus-kube-prometheus-prometheus:9090
        access: proxy
        isDefault: true

# Alertmanager Configuration
alertmanager:
  enabled: true
  
  config:
    global:
      resolve_timeout: 5m
    
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'slack'
    
    receivers:
    - name: 'slack'
      slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'  # Configure your webhook
        channel: '#alerts'
        title: 'Kubernetes Alert'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

# Node Exporter (collect node metrics)
nodeExporter:
  enabled: true

# Kube State Metrics (collect k8s object metrics)
kubeStateMetrics:
  enabled: true

# Prometheus Operator
prometheusOperator:
  enabled: true
  
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi

# Default rules (alerts)
defaultRules:
  create: true
  rules:
    alertmanager: true
    etcd: true
    configReloaders: true
    general: true
    k8s: true
    kubeApiserver: true
    kubeApiserverAvailability: true
    kubeApiserverSlos: true
    kubelet: true
    kubeProxy: true
    kubePrometheusGeneral: true
    kubePrometheusNodeRecording: true
    kubernetesApps: true
    kubernetesResources: true
    kubernetesStorage: true
    kubernetesSystem: true
    kubeScheduler: true
    kubeStateMetrics: true
    network: true
    node: true
    nodeExporterAlerting: true
    nodeExporterRecording: true
    prometheus: true
    prometheusOperator: true
```

#### Step 4: Update Chart Dependencies

```bash
cd helm/charts/kube-prometheus-stack

# Update dependencies (downloads sub-charts)
helm dependency update

# Verify dependencies
helm dependency list
```

#### Step 5: Validate the Chart

```bash
# Lint the chart
helm lint helm/charts/kube-prometheus-stack

# Dry-run installation
helm install monitoring ./helm/charts/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --dry-run --debug
```

#### Step 6: Configure Slack Webhook (Optional)

Replace `YOUR_SLACK_WEBHOOK_URL` in values.yaml:

```bash
# Create Slack incoming webhook at:
# https://api.slack.com/messaging/webhooks

# Update in values.yaml
sed -i 's|YOUR_SLACK_WEBHOOK_URL|https://hooks.slack.com/services/YOUR/WEBHOOK/URL|g' \
  helm/charts/kube-prometheus-stack/values.yaml
```

#### Step 7: Commit to Repository

```bash
git add helm/charts/kube-prometheus-stack/
git commit -m "feat: add kube-prometheus-stack helm chart with custom values"
git push origin main
```

</details>

### Option 2: Automatic Deployment (GitOps)

This repository is **automatically deployed** by Flux CD when:

1. Changes are pushed to the `main` branch
2. Flux reconciles (every 5 minutes)

**This is the first tenant deployed** (Priority 1)

### Option 3: Manual Deployment (Development)

For local testing:

```bash
# Install Helm chart directly
helm install prom ./helm/charts/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values ./helm/charts/kube-prometheus-stack/values.yaml
```

### Verify Deployment

```bash
# Check HelmRelease status
flux get helmreleases -n monitoring

# View all pods
kubectl get pods -n monitoring

# Check Prometheus
kubectl get prometheus -n monitoring

# Check Alertmanager
kubectl get alertmanager -n monitoring

# Check ServiceMonitors
kubectl get servicemonitor -A
```

---

## ⚙️ Configuration

### Helm Values

Edit `helm/charts/kube-prometheus-stack/values.yaml`:

```yaml
# Example configuration
prometheus:
  prometheusSpec:
    retention: 30d
    retentionSize: "45GB"
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi
    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: 2000m
        memory: 4Gi

grafana:
  enabled: true
  adminPassword: prom-operator  # Change in production!
  persistence:
    enabled: true
    size: 10Gi
  
alertmanager:
  enabled: true
  config:
    route:
      receiver: 'slack'
    receivers:
      - name: 'slack'
        slack_configs:
          - api_url: '<slack-webhook-url>'
            channel: '#alerts'
```

### ServiceMonitor for Custom Apps

Create ServiceMonitor to scrape your application:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

### PrometheusRule for Custom Alerts

Define custom alert rules:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-rules
  namespace: monitoring
spec:
  groups:
    - name: my-app
      interval: 30s
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status="500"}[5m]) > 0.05
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.instance }}"
```

---

## 🔐 Access

### Grafana UI

```bash
# Port-forward Grafana
kubectl port-forward -n monitoring svc/prom-grafana 3000:80

# Access via browser
open http://localhost:3000
```

**Default credentials:**
- Username: `admin`
- Password: `prom-operator` (check values.yaml)

### Prometheus UI

```bash
# Port-forward Prometheus
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-prometheus 9090:9090

# Access via browser
open http://localhost:9090
```

### Alertmanager UI

```bash
# Port-forward Alertmanager
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-alertmanager 9093:9093

# Access via browser
open http://localhost:9093
```

---

## 📊 Dashboards

### Pre-installed Dashboards

The stack includes 20+ dashboards:

| Dashboard | Description | ID |
|-----------|-------------|-----|
| Kubernetes / Compute Resources / Cluster | Overall cluster metrics | - |
| Kubernetes / Compute Resources / Namespace | Per-namespace resources | - |
| Kubernetes / Compute Resources / Pod | Per-pod metrics | - |
| Kubernetes / Networking / Cluster | Network traffic | - |
| Node Exporter / Nodes | Node-level metrics | 1860 |
| Prometheus / Overview | Prometheus stats | - |
| Flux CD | GitOps metrics | - |

### Import Custom Dashboards

```bash
# Via Grafana UI
1. Click + → Import
2. Enter dashboard ID (e.g., 1860 for Node Exporter)
3. Select Prometheus datasource
4. Click Import

# Via ConfigMap
kubectl create configmap my-dashboard \
  --from-file=dashboard.json \
  -n monitoring

# Add label for auto-discovery
kubectl label configmap my-dashboard \
  grafana_dashboard=1 \
  -n monitoring
```

### Recommended Community Dashboards

- **1860**: Node Exporter Full
- **6417**: Kubernetes Cluster Monitoring
- **7249**: Kubernetes Cluster
- **13770**: Kubernetes / Views / Pods
- **15760**: Flux CD

---

## 🔔 Alerting

### Default Alert Rules

Pre-configured alerts include:

- **Node/Pod Resource Usage**
  - CPUThrottlingHigh
  - MemoryUsageHigh
  - DiskUsageHigh

- **Kubernetes Components**
  - KubeletDown
  - KubeAPIDown
  - KubeControllerManagerDown

- **Application Health**
  - PodCrashLooping
  - PodNotReady
  - DeploymentReplicasMismatch

### Alertmanager Configuration

Configure alert routing in values.yaml:

```yaml
alertmanager:
  config:
    global:
      slack_api_url: '<webhook-url>'
    
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'slack'
      routes:
        - match:
            severity: critical
          receiver: 'pagerduty'
    
    receivers:
      - name: 'slack'
        slack_configs:
          - channel: '#alerts'
            title: '{{ .GroupLabels.alertname }}'
            text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
      
      - name: 'pagerduty'
        pagerduty_configs:
          - service_key: '<pd-integration-key>'
```

### Silence Alerts

```bash
# Via Alertmanager UI
# http://localhost:9093/#/silences

# Via CLI
amtool silence add \
  alertname=HighErrorRate \
  --comment="Planned maintenance" \
  --duration=2h
```

---

## 📈 Monitoring Best Practices

### 1. Resource Limits

Always set resource limits:

```yaml
prometheus:
  prometheusSpec:
    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: 2000m
        memory: 4Gi
```

### 2. Retention Policy

Configure based on storage:

```yaml
prometheus:
  prometheusSpec:
    retention: 30d          # Time-based
    retentionSize: "45GB"   # Size-based
```

### 3. Service Discovery

Use ServiceMonitor for automatic discovery:

```yaml
# Instead of manual scrape configs
prometheus:
  additionalServiceMonitors:
    - name: my-app
      selector:
        matchLabels:
          monitoring: "true"
```

### 4. Alert Tuning

Avoid alert fatigue:

- Set appropriate thresholds
- Use `for: 5m` to avoid transient alerts
- Group related alerts
- Define clear severity levels

---

## 🔍 Troubleshooting

### Prometheus Not Scraping Targets

```bash
# Check targets in Prometheus UI
# http://localhost:9090/targets

# Verify ServiceMonitor
kubectl get servicemonitor -A

# Check service labels match
kubectl get svc <name> -o yaml | grep labels

# Verify network policies
kubectl get networkpolicy -n <namespace>
```

### High Memory Usage

```bash
# Check Prometheus memory
kubectl top pod -n monitoring | grep prometheus

# Reduce retention
# Edit values.yaml:
retention: 15d
retentionSize: "20GB"

# Reduce scrape frequency
# In ServiceMonitor:
interval: 60s  # Instead of 30s
```

### Grafana Dashboards Not Loading

```bash
# Check Grafana pod
kubectl get pods -n monitoring | grep grafana

# View logs
kubectl logs -n monitoring deploy/prom-grafana

# Verify datasource connection
# Grafana UI → Configuration → Data Sources → Test

# Check Prometheus endpoint
kubectl get svc -n monitoring prom-kube-prometheus-stack-prometheus
```

### Alerts Not Firing

```bash
# Check Alertmanager
kubectl get alertmanager -n monitoring

# View Alertmanager logs
kubectl logs -n monitoring alertmanager-prom-kube-prometheus-stack-alertmanager-0

# Verify PrometheusRule
kubectl get prometheusrule -n monitoring

# Check alert query in Prometheus UI
# http://localhost:9090/alerts
```

### Common Issues

| Issue | Solution |
|-------|----------|
| PVC Pending | Ensure StorageClass exists |
| Prometheus OOMKilled | Increase memory limits |
| Targets down | Check ServiceMonitor selector |
| Alerts not sending | Verify Alertmanager config |
| Dashboard empty | Check time range and datasource |

---

## 🔐 Security

### Grafana Security

```yaml
# Change default password
grafana:
  adminPassword: "<strong-password>"
  
  # Enable HTTPS
  ingress:
    enabled: true
    tls:
      - secretName: grafana-tls
        hosts:
          - grafana.example.com
```

### Prometheus Security

```yaml
# Enable authentication
prometheus:
  prometheusSpec:
    web:
      httpConfig:
        http2: true
    
    # External labels for federation
    externalLabels:
      cluster: production
```

### Network Policies

Restrict access to monitoring stack:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: monitoring-policy
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: monitoring
```

---

## 📝 Development Workflow

### Making Changes

1. **Clone repository**
2. **Edit values.yaml** or add dashboards
3. **Test locally**:
   ```bash
   helm template ./helm/charts/kube-prometheus-stack
   ```
4. **Commit and push** → Flux auto-deploys!

### Adding New Metrics

1. **Expose metrics** from your application
2. **Create ServiceMonitor**
3. **Import/create Grafana dashboard**
4. **Define alert rules** (optional)

---

## 📞 Support

- 📧 Issues: [GitHub Issues](https://github.com/Cuemby/monitoring-deploy/issues)
- 📖 Prometheus Docs: [https://prometheus.io/docs/](https://prometheus.io/docs/)
- 📖 Grafana Docs: [https://grafana.com/docs/](https://grafana.com/docs/)
- 💬 Kube Prometheus Stack: [https://github.com/prometheus-community/helm-charts](https://github.com/prometheus-community/helm-charts)

---

## 📄 License

See LICENSE file for details.

---

**Made it with ❤️ by Beanters using GitOps**
