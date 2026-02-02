# monitoring-deploy Repository Index

**Prometheus + Grafana Observability Stack for Kubernetes**

This repository provides complete monitoring and observability for the Kyma CLI ecosystem using the kube-prometheus-stack Helm chart.

---

## 📋 Quick Reference

| Info | Value |
|------|-------|
| **Project** | Monitoring Stack (Tenant Repository) |
| **Stack** | kube-prometheus-stack |
| **Components** | Prometheus, Grafana, Alertmanager |
| **Chart Version** | 56.6.2 |
| **Managed By** | Flux CD (k8s-open5gs-fleet) |
| **Namespace** | `monitoring` |
| **Author** | Luis Felipe Ariza Vesga |
| **License** | Apache-2.0 |

---

## 🚀 Quick Start

```bash
# Access Grafana
kubectl port-forward -n monitoring svc/prom-grafana 3000:80
# Open: http://localhost:3000 (admin/prom-operator)

# Access Prometheus
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-prometheus 9090:9090
# Open: http://localhost:9090

# Check deployment status
flux get helmreleases -n monitoring

# View all pods
kubectl get pods -n monitoring
```

**Full Documentation**: See [README.md](README.md)

---

## 📁 Repository Structure

### Core Directories

| Directory | Purpose | Contents |
|-----------|---------|----------|
| [flux/](flux/) | Flux CD configurations | GitRepository, HelmRelease, Alerts |
| [helm/](helm/) | Helm chart | kube-prometheus-stack chart |

### Flux Configuration

**Location**: [flux/base/](flux/base/)

| File | Resource Type | Purpose |
|------|---------------|---------|
| [kustomization.yaml](flux/base/kustomization.yaml) | Kustomization | Aggregates all Flux resources |
| [monitoring-gitrepository.yaml](flux/base/monitoring-gitrepository.yaml) | GitRepository | Git source definition (5m interval) |
| [monitoring-helmrelease.yaml](flux/base/monitoring-helmrelease.yaml) | HelmRelease | Helm chart deployment config |
| [monitoring-alert.yaml](flux/base/monitoring-alert.yaml) | Alert | GitHub webhook notifications |
| [slack-notif-alert.yaml](flux/base/slack-notif-alert.yaml) | Alert | Slack notifications |

### Helm Chart

**Location**: [helm/charts/kube-prometheus-stack/](helm/charts/kube-prometheus-stack/)

| Component | Purpose | Port |
|-----------|---------|------|
| **Prometheus** | Metrics collection and storage | 9090 |
| **Grafana** | Visualization and dashboards | 3000 |
| **Alertmanager** | Alert routing and management | 9093 |
| **Node Exporter** | Node-level metrics | 9100 |
| **Kube State Metrics** | Kubernetes object metrics | 8080 |
| **Prometheus Operator** | Kubernetes-native Prometheus | - |

---

## 🎯 What's Monitored

### Metric Sources

| Source | Metrics Collected |
|--------|------------------|
| **Open5GS Network Functions** | AMF, SMF, PCF, UPF registration, sessions, PFCP |
| **Kubernetes Components** | API server, etcd, scheduler, controller-manager |
| **Nodes** | CPU, memory, disk, network, temperature |
| **Pods/Containers** | Resource usage, restarts, status |
| **Flux CD** | Reconciliations, Git sync, HelmRelease status |
| **Keptn Lifecycle** | Pre/post-deployment tasks, validation metrics |
| **Custom Exporters** | Validation metrics (ports 9092, 9093) |

### ServiceMonitor CRDs

Prometheus automatically discovers services with ServiceMonitor resources:

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
      interval: 30s
```

---

## 📊 Dashboards

### Pre-installed Dashboards (20+)

| Dashboard | Description | Focus |
|-----------|-------------|-------|
| **Kubernetes / Compute Resources / Cluster** | Overall cluster metrics | CPU, Memory, Disk |
| **Kubernetes / Compute Resources / Namespace** | Per-namespace resources | Resource usage by NS |
| **Kubernetes / Compute Resources / Pod** | Per-pod metrics | Container resources |
| **Kubernetes / Networking / Cluster** | Network traffic | Bandwidth, packets |
| **Node Exporter / Nodes** | Node-level details | Hardware metrics |
| **Prometheus / Overview** | Prometheus stats | Targets, queries, storage |
| **Flux CD** | GitOps metrics | Reconciliations, errors |

### Custom Dashboards for Kyma

Imported from [k8s-open5gs-deploy](../k8s-open5gs-deploy/):

- **5G Core Network** - Network function metrics
- **Pre-Deployment Validation Report** - Cluster readiness checks
- **Post-Deployment Validation Report** - Component health checks
- **KQI Monitoring** - Key Quality Indicators
- **5G Interfaces** - N1, N2, N3, N4 monitoring
- **Network Latency** - Inter-component latency
- **srsGNB Monitoring** - gNodeB performance
- **UE Throughput** - User equipment data rates

### Recommended Community Dashboards

| ID | Name | Purpose |
|----|------|---------|
| 1860 | Node Exporter Full | Comprehensive node metrics |
| 6417 | Kubernetes Cluster Monitoring | Multi-cluster overview |
| 7249 | Kubernetes Cluster | Cluster health |
| 13770 | Kubernetes / Views / Pods | Pod-centric view |
| 15760 | Flux CD | GitOps monitoring |

**Import via**: Grafana UI → + → Import → Enter ID

---

## 🔔 Alerting

### Default Alert Rules (40+)

**Node & Resource Alerts**:
- CPUThrottlingHigh
- MemoryUsageHigh
- DiskUsageHigh
- NodeDiskPressure
- NodeNotReady

**Kubernetes Component Alerts**:
- KubeletDown
- KubeAPIDown
- KubeControllerManagerDown
- KubeSchedulerDown

**Application Health Alerts**:
- PodCrashLooping
- PodNotReady
- DeploymentReplicasMismatch
- JobFailed

**Prometheus Alerts**:
- PrometheusTargetSyncFailure
- PrometheusConfigReloadFailed
- PrometheusNotConnectedToAlertmanager

### Alertmanager Configuration

**Location**: [helm/charts/kube-prometheus-stack/values.yaml](helm/charts/kube-prometheus-stack/values.yaml)

```yaml
alertmanager:
  config:
    route:
      receiver: 'slack'
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h

    receivers:
      - name: 'slack'
        slack_configs:
          - api_url: '<webhook-url>'
            channel: '#alerts'
            title: 'Kubernetes Alert'
```

### Silence Alerts

```bash
# Via Alertmanager UI
open http://localhost:9093/#/silences

# Via amtool CLI
amtool silence add \
  alertname=HighErrorRate \
  --comment="Planned maintenance" \
  --duration=2h
```

---

## ⚙️ Configuration

### Key Configuration Files

| File | Purpose | Location |
|------|---------|----------|
| [Chart.yaml](helm/charts/kube-prometheus-stack/Chart.yaml) | Chart metadata | helm/charts/kube-prometheus-stack/ |
| [values.yaml](helm/charts/kube-prometheus-stack/values.yaml) | Main configuration | helm/charts/kube-prometheus-stack/ |
| [Chart.lock](helm/charts/kube-prometheus-stack/Chart.lock) | Dependency lock | helm/charts/kube-prometheus-stack/ |

### values.yaml Highlights

```yaml
# Prometheus
prometheus:
  prometheusSpec:
    retention: 30d
    retentionSize: "45GB"
    storageSpec:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 50Gi

# Grafana
grafana:
  enabled: true
  adminPassword: "prom-operator"
  persistence:
    enabled: true
    size: 10Gi

# Alertmanager
alertmanager:
  enabled: true
  config:
    route:
      receiver: 'slack'
```

### Environment Overlays

**Location**: [flux/dev/](flux/dev/)

Use Flux kustomize overlays for environment-specific configs:
- `dev/` - Development environment
- `staging/` - Staging environment (future)
- `prod/` - Production environment (future)

---

## 🏗️ Architecture

### Monitoring Flow

```
┌────────────────────────────────────────────────────────────┐
│                    Application / Service                   │
│                  (exposes /metrics endpoint)               │
└────────────────────────┬───────────────────────────────────┘
                         │
                         │ Discovered by
                         ▼
┌────────────────────────────────────────────────────────────┐
│                 ServiceMonitor (CRD)                       │
│          (matches service labels, defines scrape)          │
└────────────────────────┬───────────────────────────────────┘
                         │
                         │ Prometheus Operator watches
                         ▼
┌────────────────────────────────────────────────────────────┐
│                   Prometheus Server                        │
│  • Scrapes metrics every 30s                              │
│  • Stores time series data                                │
│  • Evaluates PrometheusRule alert rules                   │
└──────────┬──────────────────────────┬──────────────────────┘
           │                          │
           │                          │ Alert triggered
           ▼                          ▼
┌────────────────────┐    ┌────────────────────────────────┐
│      Grafana       │    │       Alertmanager            │
│  • Queries Prometheus   │  • Routes alerts             │
│  • Displays dashboards  │  • Sends to Slack/PagerDuty  │
│  • 30s refresh          │  • Manages silences          │
└────────────────────┘    └───────────────────────────────┘
```

### Component Relationships

| Component | Depends On | Used By |
|-----------|------------|---------|
| **Prometheus** | - | Grafana, Alertmanager |
| **Prometheus Operator** | - | Prometheus |
| **ServiceMonitor** | Prometheus Operator | Prometheus |
| **PrometheusRule** | Prometheus Operator | Prometheus |
| **Grafana** | Prometheus | End users |
| **Alertmanager** | Prometheus | Slack, PagerDuty |
| **Node Exporter** | - | Prometheus |
| **Kube State Metrics** | - | Prometheus |

---

## 🔐 Access & Security

### Accessing Components

```bash
# Grafana (Port 3000)
kubectl port-forward -n monitoring svc/prom-grafana 3000:80

# Prometheus (Port 9090)
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-prometheus 9090:9090

# Alertmanager (Port 9093)
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-alertmanager 9093:9093
```

### Default Credentials

**Grafana**:
- Username: `admin`
- Password: `prom-operator` (configurable in values.yaml)

### Security Best Practices

1. **Change default password** in values.yaml
2. **Enable HTTPS** via Ingress with TLS
3. **Configure RBAC** for least-privilege access
4. **Use NetworkPolicies** to restrict pod communication
5. **Secure Alertmanager webhooks** with authentication

---

## 🔍 Troubleshooting

### Common Commands

```bash
# Check HelmRelease status
flux get helmreleases -n monitoring

# View all pods
kubectl get pods -n monitoring

# Check Prometheus targets
kubectl port-forward -n monitoring svc/prom-kube-prometheus-stack-prometheus 9090:9090
# Open: http://localhost:9090/targets

# View Grafana logs
kubectl logs -n monitoring deploy/prom-grafana

# Check Alertmanager status
kubectl logs -n monitoring alertmanager-prom-kube-prometheus-stack-alertmanager-0

# List ServiceMonitors
kubectl get servicemonitor -A

# List PrometheusRules
kubectl get prometheusrule -A
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| PVC Pending | No StorageClass | Create StorageClass or use hostPath |
| Prometheus OOMKilled | Insufficient memory | Increase memory limits in values.yaml |
| Targets down | ServiceMonitor selector mismatch | Check service labels match |
| Alerts not sending | Alertmanager config error | Verify webhook URL and credentials |
| Dashboard empty | Time range issue | Check time range and datasource connection |
| High memory usage | Long retention | Reduce retention or increase storage |

### Debug Checklist

```bash
# 1. Verify deployment
flux get helmreleases -n monitoring

# 2. Check pod status
kubectl get pods -n monitoring

# 3. View pod logs
kubectl logs -n monitoring <pod-name>

# 4. Check ServiceMonitors
kubectl get servicemonitor -A

# 5. Verify Prometheus targets
curl http://localhost:9090/api/v1/targets

# 6. Test Grafana datasource
# UI → Configuration → Data Sources → Test

# 7. Check Alertmanager config
kubectl get secret -n monitoring alertmanager-prom-kube-prometheus-stack-alertmanager -o yaml
```

---

## 📚 Documentation

### Main Documentation

| File | Description |
|------|-------------|
| **[README.md](README.md)** | Comprehensive deployment guide (945 lines) |
| **[LICENSE](LICENSE)** | Apache-2.0 license |

### External Documentation

| Resource | URL |
|----------|-----|
| Prometheus Docs | https://prometheus.io/docs/ |
| Grafana Docs | https://grafana.com/docs/ |
| kube-prometheus-stack Chart | https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack |
| Prometheus Operator | https://github.com/prometheus-operator/prometheus-operator |
| Alertmanager | https://prometheus.io/docs/alerting/latest/alertmanager/ |

---

## 🔗 Related Repositories

| Repository | Purpose | Relationship |
|------------|---------|--------------|
| **k8s-open5gs-vscode** | Kyma CLI source code | Manages deployment |
| **k8s-open5gs-fleet** | Multi-tenant GitOps bootstrap | Orchestrates this tenant |
| **k8s-open5gs-deploy** | Helm charts for Open5GS | Monitored by this stack |
| **keptn-deploy** | Lifecycle orchestration | Lifecycle metrics collected |
| **jaeger-deploy** | Distributed tracing | APM integration |

---

## 📊 Metrics & KPIs

### Key Metrics Exposed

**Kubernetes Metrics**:
- `kube_pod_status_phase` - Pod states (Running, Pending, Failed)
- `kube_deployment_status_replicas` - Deployment replica counts
- `kube_node_status_condition` - Node health status

**Node Metrics**:
- `node_cpu_seconds_total` - CPU usage
- `node_memory_MemAvailable_bytes` - Available memory
- `node_disk_io_time_seconds_total` - Disk I/O

**Prometheus Metrics**:
- `prometheus_tsdb_storage_blocks_bytes` - Storage size
- `prometheus_target_scrapes_total` - Scrape count
- `prometheus_rule_evaluation_failures_total` - Alert evaluation failures

**Open5GS Metrics** (from k8s-open5gs-deploy):
- `open5gs_amf_registered_ue_count` - Registered UEs
- `open5gs_smf_active_sessions` - Active PDU sessions
- `open5gs_upf_packets_total` - UPF packet counters

### Retention Policy

| Metric Type | Retention | Storage |
|-------------|-----------|---------|
| Time series | 30 days | 50Gi PVC |
| Alert history | 7 days | In-memory |
| Grafana dashboards | Persistent | 10Gi PVC |

---

## 🚦 Getting Started Workflow

1. **Prerequisites**: Fleet repository deployed, Flux CD running
2. **Automatic Deployment**: Flux deploys when changes are pushed to `main`
3. **Verify Deployment**: `flux get helmreleases -n monitoring`
4. **Access Grafana**: Port-forward and login with `admin/prom-operator`
5. **Import Dashboards**: Use community dashboard IDs or custom JSON
6. **Configure Alerts**: Edit Alertmanager config in values.yaml
7. **Monitor**: View real-time metrics and alerts

---

## 🔄 Recent Updates

### Latest Changes (January 2026)

- ✅ Deployed as first tenant (Priority 1)
- ✅ Integrated with k8s-open5gs-fleet GitOps
- ✅ Configured Slack notifications
- ✅ Added custom dashboards for Open5GS metrics
- ✅ Persistent storage enabled (30-day retention)

### Chart Version History

| Version | Date | Changes |
|---------|------|---------|
| 56.6.2 | Dec 2025 | Initial deployment with kube-prometheus-stack |

---

**Maintainer**: Luis Felipe Ariza Vesga
**License**: Apache-2.0
**Last Updated**: 2026-02-02
