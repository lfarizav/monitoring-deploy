# Helm Charts

## Available Charts

### kube-prometheus-stack

Full Prometheus + Grafana monitoring stack based on the upstream chart.

**Version**: See `Chart.yaml` for current version
**Source**: [prometheus-community/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

#### Components
- Prometheus Operator
- Prometheus Server
- Alertmanager
- Grafana
- Node Exporter
- kube-state-metrics

#### Configuration
Main configuration: `kube-prometheus-stack/values.yaml`

#### Usage
This chart is deployed via Flux HelmRelease defined in `flux/base/monitoring-helmrelease.yaml`
