## Installing the chart

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
helm install my-fluent-bit oci://dhi.io/fluent-bit-chart --version <version> \
  --set "imagePullSecrets[0].name=helm-pull-secret" \
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 4: Verify the installation

The deployment's pod should show up and running almost immediately:

```console
$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/my-fluent-bit-chart-m4l6l   1/1     Running   0          28s

NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes              ClusterIP   10.43.0.1      <none>        443/TCP    44s
service/my-fluent-bit-chart   ClusterIP   10.43.96.255   <none>        2020/TCP   28s

NAME                                   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/my-fluent-bit-chart   1         1         1       1            1           <none>          28s
```

To test that all works, get the fluent-bit pod name:

```console
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=fluent-bit-chart,app.kubernetes.io/instance=my-fluent-bit" -o jsonpath="{.items[0].metadata.name}")
```

Open a port-forwarding rule:

```console
kubectl --namespace default port-forward $POD_NAME 2020:2020
```

And from another session you can query the fluent-bit API at port 2020:

```console
curl http://127.0.0.1:2020
{"fluent-bit":{"version":"4.2.0","edition":"Community","flags":["FLB_HAVE_SYS_WAIT_H","FLB_HAVE_SIMD","FLB_HAVE_IN_STORAGE_BACKLOG","FLB_HAVE_CHUNK_TRACE","FLB_HAVE_PARSER","FLB_HAVE_RECORD_ACCESSOR","FLB_HAVE_STREAM_PROCESSOR","FLB_HAVE_UNICODE_ENCODER","FLB_EVENT_LOOP_AUTO_DISCOVERY","FLB_HAVE_TLS","FLB_HAVE_OPENSSL","FLB_HAVE_METRICS","FLB_HAVE_PROFILES","FLB_HAVE_WASM","FLB_HAVE_AWS","FLB_HAVE_AWS_CREDENTIAL_PROCESS","FLB_HAVE_SIGNV4","FLB_HAVE_SQLDB","FLB_LOG_NO_CONTROL_CHARS","FLB_HAVE_METRICS","FLB_HAVE_HTTP_SERVER","FLB_HAVE_SYSTEMD","FLB_HAVE_SYSTEMD_SDBUS","FLB_HAVE_FORK","FLB_HAVE_GMTOFF","FLB_HAVE_TIME_ZONE","FLB_HAVE_UNIX_SOCKET","FLB_HAVE_LITTLE_ENDIAN_SYSTEM","FLB_HAVE_LIBYAML","FLB_HAVE_ATTRIBUTE_ALLOC_SIZE","FLB_HAVE_PROXY_GO","FLB_HAVE_JEMALLOC","FLB_HAVE_LIBBACKTRACE","FLB_HAVE_REGEX","FLB_HAVE_UTF8_ENCODER","FLB_HAVE_LUAJIT","FLB_HAVE_ACCEPT4","FLB_HAVE_INOTIFY","FLB_HAVE_GETENTROPY","FLB_HAVE_GETENTROPY_SYS_RANDOM"]}}
```
