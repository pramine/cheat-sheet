# kubectl cheat sheet

## Delete all non-running pods

`$ kubectl delete po $(kubectl get po | grep -v Running | awk '{print $1}')`