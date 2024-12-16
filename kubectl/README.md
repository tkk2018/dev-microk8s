Delete the replicate service where 0 instances are running

```bash
microk8s kubectl get rs -n default --no-headers | awk '$2 == "0" && $3 == "0" && $4 == "0" {print $1}' | xargs -n 1 microk8s kubectl delete rs -n default
```
