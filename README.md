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
> During the `microk8s install`, it may ask to install `multipass`, a microk8s virtual machine management tool required for macOS Yosemite to run microk8s.

> [!IMPORTANT]
> For running the `microk8s` on a MAC(host machine), the following are needed:
> * multipass - A microk8s virtual machine management tool.
> * a `microk8s-vm` - An Ubuntu virtual machine where the `microk8s` runs, mananged by the `multipass`.
> 
> Although it is running inside the `microk8s-vm`, it can still be accessed from the MAC.
> 
> As you can see, the `microk8s` is actually running inside the `microk8s-vm`. Therefore, when performing custom configurations, ensure the configuration is applied to the host machine or the `microk8s-vm`.
> 
> Additionally, some of `microk8s` commands may not work on the host machine but will work in the `vm`, such as the `microk8s kubectl exec`.
> 
> For example, if there is an Ubuntu machine deployed on the node:
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
> This command fails when run on the Mac (host machine) but succeeds in the `microk8s-vm`. Not sure whether it is a bug or not.
>
> Also, if you have seen a `microk8s.<command>` pattern command, it is actually same as `microk8s <command>` but can be run in the `microk8s-vm`.
> 
> Therefore, it is recommended to always log into the `microk8s-vm` to interact with the microk8s, except for `microk8s kubectl apply` because we are not going to put the `.yaml` file in the `microk8s-vm`.
> 
> To log into the `microk8s-vm`, use the `multipass shell <name>`.
> 
> ```
> multipass shell microk8s-vm
> ```
> 
