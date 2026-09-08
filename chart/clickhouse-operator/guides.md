## Installing the chart

### Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure

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
helm install my-clickhouse-operator oci://dhi.io/clickhouse-operator-chart --version <version> \
  --set "imagePullSecrets[0].name=helm-pull-secret" \
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 4: Verify the installation

The deployment's pod should show up and running almost immediately:

```bash
$ kubectl get pods
NAME                                                      READY   STATUS    RESTARTS   AGE
my-clickhouse-clickhouse-operator-chart-64dd674c49-bbtbz   2/2     Running   0          20s
```

With the operator now running, you can create your first cluster. First create the Kubernetes resource.

```console
    cat > clickhouse-cluster.yaml << 'EOF'
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseInstallation
metadata:
  name: cluster01
spec:
  templates:
    podTemplates:
      - name: clickhouse-pod
        spec:
          imagePullSecrets:
            - name: helm-pull-secret
          containers:
            - name: clickhouse
              # Pin to a known-good Altinity Stable build (example below).
              image: dhi/clickhouse-server:25.3.6
  configuration:
    clusters:
      - name: cluster01
        layout:
          shardsCount: 1
          replicasCount: 1
        templates:
          podTemplate: clickhouse-pod
EOF
```

To then apply the resource so that it can be picked by the operator.

```console
$ kubectl apply -f clickhouse-cluster.yaml
clickhouseinstallation.clickhouse.altinity.com/cluster01 created
```

That command created a ClickHouseInstallation resource which can be also queried to check how the installation process
went.

```console
$ kubectl get ClickHouseInstallation
NAME        CLUSTERS   HOSTS   STATUS      HOSTS-COMPLETED   AGE   SUSPEND
cluster01   1          1       Completed                     44s

$ kubectl describe ClickHouseInstallation/cluster01
Name:         cluster01
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  clickhouse.altinity.com/v1
Kind:         ClickHouseInstallation
Metadata:
  Creation Timestamp:  2025-10-09T13:21:11Z
  Finalizers:
    finalizer.clickhouseinstallation.altinity.com
  Generation:        1
  Resource Version:  411144
  UID:               e2804722-530a-47cc-9939-0669aa290450
Spec:
  Configuration:
    Clusters:
      Layout:
        Replicas Count:  1
        Shards Count:    1
      Name:              cluster01
      Templates:
        Pod Template:  clickhouse-pod
  Templates:
    Pod Templates:
      Name:  clickhouse-pod
      Spec:
        Containers:
          Image:  dhi/clickhouse-server:25.3.6
          Name:   clickhouse
        Image Pull Secrets:
          Name:  helm-pull-secret
Status:
  Chop - Commit:   4e069d29e4ae78a8b78ac74e86c1e090f87bf460
  Chop - Date:     2025-10-02-21:28:00
  Chop - Ip:       10.244.1.108
  Chop - Version:  0.25.4
  Clusters:        1
  Endpoint:        clickhouse-cluster01.default.svc.cluster.local
  Endpoints:
    clickhouse-cluster01.default.svc.cluster.local
  Fqdns:
    chi-cluster01-cluster01-0-0.default.svc.cluster.local
  Hosts:  1
  Pods:
    chi-cluster01-cluster01-0-0-0
  Shards:   1
  Status:   Completed
  Task ID:  auto-68030f68-93e4-4910-b7a8-fbeaaaee18b9
  Task I Ds Completed:
    auto-68030f68-93e4-4910-b7a8-fbeaaaee18b9
  Task I Ds Started:
    auto-6af8dd63-36a4-4bae-8795-894c839f635c
Events:
  Type   Reason                  Age   From                 Message
  ----   ------                  ----  ----                 -------
  Info   ReconcileStarted        105s  clickhouse-operator  reconcile started, task id: auto-6af8dd63-36a4-4bae-8795-894c839f635c
  Info   CreateCompleted         104s  clickhouse-operator  Create ConfigMap default/chi-cluster01-common-configd
  Info   CreateCompleted         103s  clickhouse-operator  Create ConfigMap default/chi-cluster01-common-usersd
  Info   CreateCompleted         103s  clickhouse-operator  Create ConfigMap default/chi-cluster01-deploy-confd-cluster01-0-0
  Info   CreateCompleted         102s  clickhouse-operator  OK Create Service: default/chi-cluster01-cluster01-0-0
  Error  UpdateFailed            102s  clickhouse-operator  Update Service: default/chi-cluster01-cluster01-0-0 failed with error: just recreate the service in case of service type change ''=>'ClusterIP'
  Info   CreateStarted           102s  clickhouse-operator  Update StatefulSet(default/chi-cluster01-cluster01-0-0) - started
  Info   CreateStarted           101s  clickhouse-operator  Create StatefulSet: default/chi-cluster01-cluster01-0-0 - started
  Info   UpdateInProgress        101s  clickhouse-operator  Update StatefulSet(default/chi-cluster01-cluster01-0-0) switch from Update to Recreate
  Info   UpdateCompleted         98s   clickhouse-operator  Update ConfigMap default/chi-cluster01-common-usersd
  Info   CreateCompleted         85s   clickhouse-operator  Create StatefulSet: default/chi-cluster01-cluster01-0-0 - completed
  Info   UpdateCompleted         85s   clickhouse-operator  Update ConfigMap default/chi-cluster01-common-configd
  Info   ProgressHostsCompleted  75s   clickhouse-operator  [now: 2025-10-09 13:21:41.25382659 +0000 UTC m=+360.019283289] ProgressHostsCompleted: 1 of 1
  Info   ReconcileCompleted      75s   clickhouse-operator  Reconcile Host completed. Host: 0-0 ClickHouse version running: 25.3.6[25.3.6/parsed from the tag: '25.3.6']
  Error  UpdateFailed            75s   clickhouse-operator  Update Service: default/clickhouse-cluster01 failed with error: just recreate the service in case of service type change ''=>'ClusterIP'
  Info   CreateCompleted         75s   clickhouse-operator  OK Create Service: default/clickhouse-cluster01
  Info   ReconcileInProgress     74s   clickhouse-operator  remove items scheduled for deletion
  Info   UpdateCompleted         74s   clickhouse-operator  Update ConfigMap default/chi-cluster01-common-configd
  Info   ReconcileInProgress     72s   clickhouse-operator  add CHI to monitoring
  Info   UpdateCompleted         71s   clickhouse-operator  Update ConfigMap default/chi-cluster01-common-usersd
  Info   ReconcileCompleted      70s   clickhouse-operator  reconcile completed successfully, task id: auto-6af8dd63-36a4-4bae-8795-894c839f635c
```

And the cluster pod should be up and running.

```console
$ kubectl get pods
NAME                                                       READY   STATUS    RESTARTS   AGE
chi-cluster01-cluster01-0-0-0                              1/1     Running   0          2m50s
my-clickhouse-clickhouse-operator-chart-64dd674c49-bbtbz   2/2     Running   0          8m34s
```
