## About this Helm chart

This is a Thnaos Docker Hardened Helm chart built from the upstream Thanos Helm chart and using a hardened configuration
with Docker Hardened Images.

The following Docker Hardened Images are used in this Helm chart:

- `dhi/thanos`
- `dhi/configmap-reload`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://github.com/stevehipwell/helm-charts/blob/main/charts/thanos/README.md](https://github.com/stevehipwell/helm-charts/blob/main/charts/thanos/README.md)

## About Thanos

Thanos is an open-source project that enhances Prometheus by providing high availability, global querying, and long-term
storage capabilities. It integrates seamlessly with existing Prometheus deployments, allowing organizations to scale
their monitoring infrastructure efficiently.

Thanos enables querying across multiple Prometheus servers and clusters, offering a unified view of metrics. It supports
object storage solutions like AWS S3, GCP, and Azure for indefinite retention of metrics. By maintaining compatibility
with Prometheus's query API, Thanos allows the use of existing tools like Grafana.

For more details, visit https://thanos.io/.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with zero-known CVEs, include signed provenance, and come with a complete Software Bill of
Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly into
existing Docker workflows.

## Trademarks

This listing is prepared by Docker. All third-party product names, logos, and trademarks are the property of their
respective owners and are used solely for identification. Docker claims no interest in those marks, and no affiliation,
sponsorship, or endorsement is implied.
