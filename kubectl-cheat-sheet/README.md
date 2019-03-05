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

## Know Istio version

`kubectl get po istio-pilot-xxx-xxx -n istio-system --template='{{(index (index .spec.containers 0).image)}}{{"\n"}}' | awk -F ":" '{ print $2 }'`
