## About this Helm chart

This is a Prometheus Helm chart built from the upstream Node Exporter Helm chart and using a hardened configuration with
Docker Hardened Images.

The following Docker Hardened Helm charts are used in this Helm chart:

- `dhi/alertmanager-chart`
- `dhi/kube-state-metrics-chart`
- `dhi/node-exporter-chart`
- `dhi/pushgateway-chart`

The following Docker Hardened Images are used in this Helm chart:

- `dhi/prometheus`
- `dhi/prometheus-config-reloader`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/README.md](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/README.md)

## About Prometheus

Prometheus is an open-source monitoring and alerting system designed for reliability and scalability. It collects
metrics from configured targets at specified intervals, evaluates rule-based conditions, and triggers alerts when
thresholds are crossed.

Metrics in Prometheus are stored as time series data, with each data point tagged by a timestamp and optional
labels—key-value pairs that provide context such as host, service, or environment. Prometheus supports a powerful query
language called PromQL, which lets you aggregate, filter, and analyze metric data in real time.

For more details, visit https://prometheus.io/docs/.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with near-zero known CVEs, include signed provenance, and come with a complete Software Bill
of Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly
into existing Docker workflows.

## Trademarks

Kubernetes® and Prometheus® are all trademarks of the Linux Foundation. All rights in the marks are reserved to the
Linux Foundation. Any use by Docker is for referential purposes only and does not indicate sponsorship, endorsement, or
affiliation.
