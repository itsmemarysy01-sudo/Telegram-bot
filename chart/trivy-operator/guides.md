## Installing the chart

### Prerequisites

- Kubernetes 1.21+ (recommended 1.30+)
- Helm 3.6+ (recommended 3.7+)

### Installation steps

All examples in this guide use the public chart and images. If you've mirrored the repository for your own use (for
example, to your Docker Hub namespace), update your commands to reference the mirrored chart instead of the public one.

For example:

- Public chart: `dhi.io/<repository>:<tag>`
- Mirrored chart: `<your-namespace>/dhi-<repository>:<tag>`

For more details about customizing the chart to reference other images, see the
[documentation](https://docs.docker.com/dhi/how-to/customize/).

#### Step 1: Optional. Mirror the Helm chart and/or its images to your own registry

To optionally mirror a chart to your own third-party registry, you can follow the instructions in
[How to mirror an image ](https://docs.docker.com/dhi/how-to/mirror/) for either the chart, the image, or both.

The same `regctl` tool that is used for mirroring container images can also be used for mirroring Helm charts, as Helm
charts are OCI artifacts.

For example:

```console
 regctl image copy \
     "${SRC_CHART_REPO}:${TAG}" \
     "${DEST_REG}/${DEST_CHART_REPO}:${TAG}" \
     --referrers \
     --referrers-src "${SRC_ATT_REPO}" \
     --referrers-tgt "${DEST_REG}/${DEST_CHART_REPO}" \
     --force-recursive
```

#### Step 2: Create a Kubernetes secret for pulling images

The Docker Hardened Images that the chart uses require authentication. To allow your Kubernetes cluster to pull those
images, you need to create a Kubernetes secret with your Docker Hub credentials or with the credentials for your own
registry.

Follow the [authentication instructions for DHI in Kubernetes](https://docs.docker.com/dhi/how-to/k8s/#authentication).

For example:

```console
kubectl create secret docker-registry helm-pull-secret \
  --docker-server=dhi.io \
  --docker-username=<Docker username> \
  --docker-password=<Docker token> \
  --docker-email=<Docker email>
```

#### Step 3: Install the Helm chart

To install the chart, use `helm install`. Make sure you use `helm login` to log in before running `helm install`.
Optionally, you can also use the `--dry-run` flag to test the installation without actually installing anything.

```console
helm install my-trivy-operator oci://dhi.io/trivy-operator-chart --version <version> \
  --set "image.pullSecrets[0].name=helm-pull-secret" \
  --set "trivy.image.imagePullSecret=helm-pull-secret" \
  --set "nodeCollector.imagePullSecret=helm-pull-secret"
```

Note: Trivy Operator configures image pull secrets separately for each image. `image.pullSecrets` applies to the
operator itself, `trivy.image.imagePullSecret` applies to the Trivy scanner used in scan jobs, and
`nodeCollector.imagePullSecret` applies to the node-collector used in node/infrastructure scans.

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 4: Verify the installation

The operator pod should be running within a few seconds:

```bash
$ kubectl get all
NAME                                             READY   STATUS    RESTARTS   AGE
pod/test-trivy-operator-chart-74f47f4b6f-szww6   1/1     Running   0          32s

NAME                                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/test-trivy-operator-chart   ClusterIP   None         <none>        80/TCP    32s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-trivy-operator-chart   1/1     1            1           32s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/test-trivy-operator-chart-74f47f4b6f   1         1         1       32s
```

Once running, the operator will begin scanning workloads in your cluster and producing security reports as Kubernetes
custom resources. You can list them with:

```console
kubectl get vulnerabilityreports -A
kubectl get configauditreports -A
```

## Using the built-in Trivy server

By default, each scan job runs its own Trivy process and downloads the vulnerability database independently. For
clusters with many workloads, you can enable the built-in Trivy server, which caches the vulnerability database once and
serves all scan jobs from a single StatefulSet:

```console
helm install my-trivy-operator oci://dhi.io/trivy-operator-chart --version <version> \
  --set "image.pullSecrets[0].name=helm-pull-secret" \
  --set "trivy.image.imagePullSecret=helm-pull-secret" \
  --set "nodeCollector.imagePullSecret=helm-pull-secret" \
  --set "operator.builtInTrivyServer=true"
```

## Network requirements

Trivy Operator requires outbound HTTPS access to the following hosts at scan time:

- `ghcr.io` — vulnerability database (`trivy-db`) and misconfiguration checks (`trivy-checks`)
- `mirror.gcr.io` — fallback mirror for the vulnerability database

In air-gapped environments, you can mirror these OCI artifacts to an internal registry and configure Trivy to use it:

```console
helm install my-trivy-operator oci://dhi.io/trivy-operator-chart --version <version> \
  --set "image.pullSecrets[0].name=helm-pull-secret" \
  --set "trivy.image.imagePullSecret=helm-pull-secret" \
  --set "nodeCollector.imagePullSecret=helm-pull-secret" \
  --set "trivy.dbRepository=<your-registry>/aquasecurity/trivy-db" \
  --set "trivy.javaDbRepository=<your-registry>/aquasecurity/trivy-java-db"
```
