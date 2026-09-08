## About this Helm chart

This is a Trivy Operator Helm chart built from the upstream Trivy Operator Helm chart and using a hardened configuration
with Docker Hardened Images.

The following Docker Hardened Images are used in this Helm chart:

- `dhi/trivy-operator`
- `dhi/trivy`
- `dhi/node-collector`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://aquasecurity.github.io/trivy-operator/latest/](https://aquasecurity.github.io/trivy-operator/latest/)

## About Trivy Operator

Trivy Operator is a Kubernetes-native security scanner that continuously audits your cluster and produces security
reports as Kubernetes custom resources. It automatically scans container images for known vulnerabilities, audits
workload configurations against security best practices, and checks cluster nodes against the CIS Kubernetes Benchmark.

Reports are stored as first-class Kubernetes objects (`VulnerabilityReport`, `ConfigAuditReport`,
`ClusterInfraAssessmentReport`, and others), making them queryable with standard Kubernetes tooling and easy to
integrate into existing workflows.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with near-zero known CVEs, include signed provenance, and come with a complete Software Bill
of Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly
into existing Docker workflows.

## Trademarks

Trivy® is a trademark of Aqua Security. All rights in the mark are reserved to Aqua Security. Any use by Docker is for
referential purposes only and does not indicate sponsorship, endorsement, or affiliation.
