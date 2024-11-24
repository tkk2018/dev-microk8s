> [!NOTE]
> If using the Multipass + MicroK8s combo, it's important to be aware of the relationship between the host machine and `microk8s-vm`.
> 
> The `microk8s` has a [built-in registry](https://microk8s.io/docs/registry-built-in). It is Distribution API V2 compliant.
>
> If the registry supports the Distribution API V2, it should be used instead of performing this locally.

## List the images

```
microk8s ctr images ls
```
