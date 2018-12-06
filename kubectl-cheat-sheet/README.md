# kubectl cheat sheet

## view kubernetes context and login/password

`$ kubectl config view --minify`

## Delete all non-running pods

`$ kubectl delete po $(kubectl get po | grep -v Running | awk '{print $1}') | grep -v NAME`