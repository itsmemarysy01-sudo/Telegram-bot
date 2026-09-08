## Installing the chart

### Prerequisites

- Kubernetes 1.25+

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
helm install my-neo4j oci://dhi.io/charts/neo4j-chart --version <version> \
  --set "image.imagePullSecrets[0]=helm-pull-secret"
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 4: Verify the installation

Neo4j pod(s) should show up and running almost immediately:

```console
$ kubectl get all
NAME          READY   STATUS    RESTARTS   AGE
pod/neo4j-0   1/1     Running   0          1m5s

NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                        AGE
service/kubernetes       ClusterIP      10.96.0.1       <none>        443/TCP                                        55d
service/neo4j            ClusterIP      10.98.98.98     <none>        7687/TCP,7474/TCP                              1m5s
service/neo4j-admin      ClusterIP      10.110.157.82   <none>        7687/TCP,7474/TCP                              1m5s
service/neo4j-lb-neo4j   LoadBalancer   10.98.198.42    localhost     7474:30358/TCP,7473:30376/TCP,7687:30306/TCP   1m5s

NAME                     READY   AGE
statefulset.apps/neo4j   1/1     1m5s
```
