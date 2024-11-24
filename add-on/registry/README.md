# Built-in registry

[Official Docs](https://microk8s.io/docs/registry-built-in)

The `registry` is compliant with the [Distribution HTTP API V2](https://distribution.github.io/distribution/spec/api/).

Tested machine information:

```bash
$ sw_vers
ProductName:		macOS
ProductVersion:		14.4
BuildVersion:		23E214

% multipass --version
multipass   1.14.1+mac
multipassd  1.14.1+mac

$ microk8s version
MicroK8s v1.28.15 revision 7407
```

### Enable the add-on

```bash
microk8s enable registry
```

The registry is deployed in kubernetes on the same machine.

You can check the enabled registry by using `microk8s kubectl get all`.

```bash
$ microk8s kubectl get all --all-namespaces
NAMESPACE            NAME                                             READY   STATUS    RESTARTS      AGE
container-registry   pod/registry-xxxxxxxxxx-aaaaa                    1/1     Running   3 (27s ago)   17d
...

NAMESPACE            NAME                                TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                  AGE
container-registry   service/registry                    NodePort       10.152.183.135   <none>         5000:32000/TCP           17d
...

NAMESPACE            NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
container-registry   deployment.apps/registry                    1/1     1            1           17d
...

NAMESPACE            NAME                                                   DESIRED   CURRENT   READY   AGE
container-registry   replicaset.apps/registry-xxxxxxxxxx                    1         1         1       17d
...
```

Thus, it can be accessed through `localhost:32000`.

> [!IMPORTANT]
> When using the Multipass + MicroK8s combo, the `microk8s-vm` IP address can be obtained by using the `multipass info`.
>
> ```bash
> $ multipass info
> Name:           microk8s-vm
> State:          Running
> Snapshots:      0
> IPv4:           192.168.64.1
> ...
> ```
>
> Therefore, it can be accessed through `192.168.64.1:32000` from the host machine.

__For managing the registry's images, check the `microk8s ctr images` command__ or the Distribution HTTP API V2.

## Docker and Podman

To push images to the `registry` using either Docker or Podman, set the `localhost:32000` as an insecure registry. If using the Multipass + MicroK8s combo and the Docker or Podman is installed in the host machine, uses the IP address of `microk8s-vm` instead.<sup>[\[1\]](https://microk8s.io/docs/registry-built-in#what-if-microk8s-runs-inside-a-vm)</sup><sup>[\[2\]](https://discuss.kubernetes.io/t/how-to-use-the-built-in-registry/11274)</sup>

> [!NOTE]
> Whether it is a native Ubuntu host or the Multipass + MicroK8s combo, the image address for pulling from the container is `localhost:32000` because the process is happening on the `microK8s` machine.

## Access to the Pod

> [!NOTE]
> This should not be required, just use the `microk8s ctr images` or the Distribution V2 API. This is just additional knowledge.

### Log in

> [!IMPORTANT]
> If using the Multipass + Microk8s combo, the `microk8s kubectl exec` command must be run within the `microk8s-vm`.

Get the correct pod's name

```bash
$ microk8s kubectl get pods -n container-registry
NAME                        READY   STATUS    RESTARTS      AGE
registry-xxxxxxxxxx-aaaaa   1/1     Running   3 (29m ago)   17d
```

The `registry-xxxxxxxxxx-aaaaa` is the name of the pod where the registry is installed.

Log in to the machine using the pod's name

```bash
$ microk8s kubectl exec -it registry-xxxxxxxxxx-aaaaa -n container-registry -- /bin/sh
/ #
/ # registry --version
registry github.com/docker/distribution v2.8.1+unknown
```

## Update the registry image version

The registry image version can be updated by upgrading the microk8s version. However, whether this includes the latest registry version depends on the MicroK8s team’s decision to roll it out with the MicroK8s version. 

Alternatively, it can be done manually by following the steps below.

Get the deployment details of the `registry`:

```bash
microk8s kubectl describe deployment registry -n container-registry
```

Set the image version:

```bash
# Set the image version
# microk8s kubectl set image deployment/<deployment-name> -n <namespace> <container-name>=<image-name>:<tag>
microk8s kubectl set image deployment/registry -n container-registry registry=registry:2.8.3
```

Roll out the container:

```bash
microk8s kubectl rollout restart deploy registry -n container-registry
```

## Distribution HTTP API V2<sup>[\[1\]](https://tech.michaelaltfield.net/2024/09/03/container-download-curl-wget/)</sup>

Let's say the stack is a Multipass + MicroK8s combo, and the IP address of the `microk8s-vm` is `192.168.64.1`. It has an image `node-simple-server:latest` pushed to the registry.

To get all images

```bash
$ curl http://192.168.64.1:32000/v2/_catalog
{"repositories":["node-simple-server"]}
```

To get the image's tags

```bash
curl http://192.168.64.1:32000/v2/node-simple-server/tags/list
{"name":"node-simple-server","tags":["latest"]}
```

This two can be done in one line with the following command<sup>[\[1\]](https://github.com/canonical/microk8s/issues/382#issuecomment-488229576)</sup>, but first, it requires the `jq` package:

```bash
brew install jq
```

Then

```bash
$ curl -k -s -X GET http://192.168.64.1:32000/v2/_catalog \
| jq '.repositories[]'  \
| sort \
| xargs -I _ curl -s -k -X GET http://192.168.64.1:32000/v2/_/tags/list
{"name":"node-simple-server","tags":["latest"]}
```

To get the image's manifest

> [!NOTE]
> It is recommended to use `GET` intead of `HEAD` because if there is any error, the error message will be available in the response body.

> [!IMPORTANT]
> When to using the `registry:2.8.1`, there is an [issue](https://github.com/docker/hub-feedback/issues/2410) where it does not correctly follow the `Accept` header. Therefore, it requires trial and error to determine the correct accepted format.

The available headers are:<sup>[\[1\]](https://stackoverflow.com/a/69384703/16027098)</sup>

```
application/vnd.docker.distribution.manifest.v2+json
application/vnd.docker.distribution.manifest.list.v2+json
application/vnd.oci.image.manifest.v1+json
application/vnd.oci.image.index.v1+json
```

In our case, use `application/vnd.oci.image.index.v1+json`.

```bash
$ curl -H "Accept: application/vnd.oci.image.index.v1+json" -i http://192.168.64.1:32000/v2/node-simple-server/manifests/latest
HTTP/1.1 200 OK
Content-Length: 1609
Content-Type: application/vnd.oci.image.index.v1+json
Docker-Content-Digest: sha256:xxxxx
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:xxxxx"
X-Content-Type-Options: nosniff
Date: Sun, 24 Nov 2024 07:41:08 GMT

{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    ...
  ]
{
```

The `Docker-Content-Digest` is the manifest.

To delete:<sup>[\[1\]](https://betterprogramming.pub/cleanup-your-docker-registry-ef0527673e3a)</sup>

```bash
$ curl -v \
-H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
-X DELETE 192.168.64.1:32000/v2/node-simple-server/manifests/sha256:xxxxx
HTTP/1.1 202 Accepted
...
```
