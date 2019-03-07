# kubectl cheat sheet

## view kubernetes context and login/password

`$ kubectl config view --minify`

## Delete all non-running pods

`$ kubectl delete po $(kubectl get po --no-headers | grep -v Running | awk '{print $1}')`

or more natively:

`$ kubectl delete $(kubectl get pod --no-headers --field-selector=status.phase!=Running -o name )`

## Run a shell in a new pod 

Useful in order to examine the kubernetes environment from the inside:

`alias kshell='kubectl run -it shell --image giantswarm/tiny-tools --restart Never --rm -- sh'`

## Delete pod stucked in Terminating status

`$ kubectl delete po PODNAME --force --grace-period 0`

## Know Istio version

```
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' -n istio-system | grep pilot)
$ kubectl get po $POD_NAME -n istio-system --template='{{(index (index .spec.containers 0).image)}}{{"\n"}}' | awk -F ":" '{ print $2 }'
```
