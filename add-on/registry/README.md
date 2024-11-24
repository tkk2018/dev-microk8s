# Built-in registry

[Official Docs](https://microk8s.io/docs/registry-built-in)

The machine information:

```
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

You can check the enabled registry by using `microk8s kubectl get all`.

```bash
$ microk8s kubectl get all --all-namespaces
NAMESPACE            NAME                                             READY   STATUS    RESTARTS      AGE
container-registry   pod/registry-6c9fcc695f-p55r2                    1/1     Running   3 (27s ago)   17d
...

NAMESPACE            NAME                                TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                  AGE
container-registry   service/registry                    NodePort       10.152.183.133   <none>         5000:32000/TCP           17d
...

NAMESPACE            NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
container-registry   deployment.apps/registry                    1/1     1            1           17d
...

NAMESPACE            NAME                                                   DESIRED   CURRENT   READY   AGE
container-registry   replicaset.apps/registry-6c9fcc695f                    1         1         1       17d
...
```

Thus, it can be accessed through `localhost:32000`.

> [!IMPORTANT]
> When using the Multipass + MicroK8s combo, the `microk8s-vm` IP address can be obtained by using the `multipass info`.
>
> ```
> $ multipass info
> Name:           microk8s-vm
> State:          Running
> Snapshots:      0
> IPv4:           192.168.64.1
> ...
> ```
>
> Therefore, it can be accessed through `192.168.64.1:32000` from the host machine.

The `registry` is compliant with the [Distribution API V2](https://distribution.github.io/distribution/spec/api/). For managing the registry's `images`, check the `microk8s ctr images` command.

### Docker and Podman

To push images to the `registry` using either Docker or Podman, set the `localhost:32000` as an insecure registry. 

If using the Multipass + MicroK8s combo and the Docker or Podman is installed in the host machine, check the IP address of `microk8s-vm` by using the `multipass info` command.

### Log in to the pod

> [!NOTE]
> This should not be required, just use the `microk8s ctr images` and the Distribution V2 API. This is just additional knowledge.

> [!IMPORTANT]
> If using the Multipass + Microk8s combo, the `microk8s kubectl exec` command must be run within the `microk8s-vm`.

Get the correct pod's name

```
$ microk8s kubectl get pods -n container-registry
NAME                        READY   STATUS    RESTARTS      AGE
registry-xxxxxxxxxx-xxxxx   1/1     Running   3 (29m ago)   17d
```

The `registry-xxxxxxxxxx-xxxxx` is the name of the pod where the registry is installed.

Log in to the machine using the pod's name

```bash
$ microk8s kubectl exec -it registry-xxxxxxxxxx-xxxxx -n container-registry -- /bin/sh
/ #
/ # registry --version
registry github.com/docker/distribution v2.8.1+unknown
```

### Update the registry image version

The registry image version can be updated by upgrading the microk8s version. However, whether this includes the latest registry version depends on the MicroK8s team’s decision to roll it out with the MicroK8s version. 

Alternatively, it can be done manually by following the steps below.

Get the deployment details of the `registry`:

```
microk8s kubectl describe deployment registry -n container-registry
```

Set the image version:

```
# Set the image version
# microk8s kubectl set image deployment/<deployment-name> -n <namespace> <container-name>=<image-name>:<tag>
microk8s kubectl set image deployment/registry -n container-registry registry=registry:2.8.3
```

Roll out the container:
```
microk8s kubectl rollout restart deploy registry -n container-registry
```
