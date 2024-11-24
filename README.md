## Install

### Installing on MAC

[Official docs](https://microk8s.io/docs/install-macos)

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

> [!NOTE]
> For running the `microk8s` on a MAC (host machine), the following are needed:
> * `multipass` - Microk8s virtual machine management tool.
> * `microk8s-vm` - Ubuntu virtual machine where the `microk8s` runs, mananged by the `multipass`.
> 
> During the `microk8s install`, it may ask to install `multipass`, a microk8s virtual machine management tool required for macOS Yosemite to run microk8s.
> 
> Therefore, the stack is MAC (host machine) -> multipass -> microk8s-vm -> microk8s -> kubernetes.
>
> Although the `microk8s` is actually running inside the `microk8s-vm`, it can still be accessed from the host machine.

> [!IMPORTANT]
> When using the Multipass + Microk8s combo:
>
> When changing certain configurations, please ensure that the changes should be done on the host machine or the `microk8s-vm`.
> 
> Additionally, some of `microk8s` commands may not work on the host machine but will work in the `microk8s-vm`, such as the `microk8s kubectl exec`.
> 
> For example, if there is an Ubuntu machine deployed on the node,
> 
> ```
> microk8s kubectl get pods    
> NAME                               READY   STATUS    RESTARTS        AGE
> ubuntu-deployment-12345678-xyz12   1/1     Running   1 (1h ago)      2h
> ```
> 
> and you want to log into the machine, you would do:
> 
> ```
> microk8s kubectl exec -it ubuntu-deployment-12345678-xyz12 -- /bin/bash
> ```
> 
> This command fails when run on the host machine but succeeds in the `microk8s-vm`.
> 
> ```
> error: exec [POD] [COMMAND] is not supported anymore. Use exec [POD] -- [COMMAND] instead
> See 'kubectl exec -h' for help and examples
> ```
> 
> Not sure whether it is a bug.
>
> Also, if you have seen a `microk8s.<command>` pattern command, it is actually same as `microk8s <command>` but can be run in the `microk8s-vm`.
> 
> Therefore, it is recommended to log into the `microk8s-vm` to interact with the `microk8s`.
> 
> ```
> multipass shell microk8s-vm
> ```
>
> Except for certain processes such as:
> * `microk8s kubectl apply` - because the `.yaml` files normally sit on the host machine.
> * Docker/Podman-related tasks - because both are normally installed on the host machine.
