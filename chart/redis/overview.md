## About this Helm chart

This is a Redis Docker Hardened Helm chart built from the upstream Redis Helm chart and using a hardened configuration
with Docker Hardened Images.

The following Docker Hardened Images are used in this Helm chart:

- `dhi/redis`
- `dhi/redis-exporter`
- `dhi/bash`
- `dhi/kubectl`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://redis.io/docs/latest/operate/kubernetes/deployment/helm](https://redis.io/docs/latest/operate/kubernetes/deployment/helm/)

## About Redis

Redis is the world's fastest data platform. It provides cloud and on-prem solutions for caching, vector search, and
NoSQL databases that seamlessly fit into any tech stack—making it simple for digital customers to build, scale, and
deploy the fast apps our world runs on.

For more details, visit https://redis.io/.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published near-zero known CVEs, include signed provenance, and come with a complete Software Bill of
Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly into
existing Docker workflows.

## Trademarks

Redis® is a registered trademark of Redis Ltd. Any rights therein are reserved to Redis Ltd. Any use by Docker is for
referential purposes only and does not indicate any sponsorship, endorsement, or affiliation between Redis Ltd.
