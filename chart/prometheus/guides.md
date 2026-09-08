## Installing the chart

### Prerequisites

- Kubernetes 1.17+ (recommended 1.30+)
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
helm install my-prometheus oci://dhi.io/prometheus-chart --version <version> \
  --set "imagePullSecrets[0].name=helm-pull-secret" \
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 4: Verify the installation

The deployment's pods should show up and running almost immediately:

```bash
$ kubectl get all
NAME                                                 READY   STATUS    RESTARTS   AGE
pod/test-alertmanager-0                              1/1     Running   0          32s
pod/test-kube-state-metrics-chart-68f8dc76f8-whqs4   1/1     Running   0          32s
pod/test-prometheus-chart-server-6758c45586-t22fh    2/2     Running   0          32s
pod/test-prometheus-node-exporter-srsdv              1/1     Running   0          32s
pod/test-prometheus-pushgateway-7dd4d54c9-bxp4k      1/1     Running   0          32s

NAME                                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                      ClusterIP   10.43.0.1      <none>        443/TCP    54s
service/test-alertmanager               ClusterIP   10.43.38.129   <none>        9093/TCP   32s
service/test-alertmanager-headless      ClusterIP   None           <none>        9093/TCP   32s
service/test-kube-state-metrics-chart   ClusterIP   10.43.97.216   <none>        8080/TCP   32s
service/test-prometheus-chart-server    ClusterIP   10.43.56.61    <none>        80/TCP     32s
service/test-prometheus-node-exporter   ClusterIP   10.43.28.46    <none>        9100/TCP   32s
service/test-prometheus-pushgateway     ClusterIP   10.43.3.226    <none>        9091/TCP   32s

NAME                                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/test-prometheus-node-exporter   1         1         1       1            1           kubernetes.io/os=linux   32s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-kube-state-metrics-chart   1/1     1            1           32s
deployment.apps/test-prometheus-chart-server    1/1     1            0           32s
deployment.apps/test-prometheus-pushgateway     1/1     1            1           32s

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/test-kube-state-metrics-chart-68f8dc76f8   1         1         1       32s
replicaset.apps/test-prometheus-chart-server-6758c45586    1         1         1       32s
replicaset.apps/test-prometheus-pushgateway-7dd4d54c9      1         1         1       32s

NAME                                 READY   AGE
statefulset.apps/test-alertmanager   1/1     32s


```
