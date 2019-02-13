# kubectl cheat sheet

## view kubernetes context and login/password

`$ kubectl config view --minify`

## Delete all non-running pods

`$ kubectl delete po $(kubectl get po --no-headers | grep -v Running | awk '{print $1}')`

## Run a shell in a new pod 

Useful in order to examine the kubernetes environment from the inside:

`alias kshell='kubectl run -it shell --image giantswarm/tiny-tools --restart Never --rm -- sh'`
