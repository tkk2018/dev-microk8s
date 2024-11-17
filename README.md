## Install

### Installing on MAC

> [!IMPORTANT]
> For running the `microk8s` stack on a MAC(host machine), the following are needed:
> * multipass - A microk8s virtual machine management tool.
> * a `microk8s-vm` - An Ubuntu virtual machine where the `microk8s` runs, mananged by the `multipass`.
> 
> Although it is running inside the `microk8s-vm`, it can still be accessed from the MAC.
> 
> As you can see, the `microk8s` is actually running inside the `microk8s-vm`. Therefore, when performing custom configurations, ensure the configuration is applied to the MAC or the `microk8s-vm`.
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
> Therefore, it is recommended to always log into the `vm`, except for `microk8s kubectl apply` because we are not going to put the `.yaml` file in the `vm`.
> 
> To log into the `microk8s-vm`, use the `multipass shell <name>`.
> 
> ```
> multipass shell microk8s-vm
> ```
> 
