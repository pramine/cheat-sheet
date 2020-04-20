# kubectl cheat sheet

## view kubernetes context and login/password

`$ kubectl config view --minify`

## Delete all non-running pods

`$ kubectl delete po $(kubectl get po --no-headers | grep -v Running | awk '{print $1}')`

or more natively:

`$ kubectl delete po $(kubectl get pod --no-headers --field-selector=status.phase!=Running )`

## Run a shell in a new pod 

Useful in order to examine the kubernetes environment from the inside:

`alias kshell='kubectl run -it shell --image giantswarm/tiny-tools --restart Never --rm -- sh'`

## Delete pod stucked in Terminating status

`$ kubectl delete po PODNAME --force --grace-period 0`

## Retrieve istio ingress (load balancer) hostname

`$ kubectl get svc istio-ingressgateway -n istio-system -o json | jq .status.loadBalancer.ingress[0]`

## Know Istio version

```
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' -n istio-system | grep pilot | head -1)
$ kubectl get po $POD_NAME -n istio-system --template='{{(index (index .spec.containers 0).image)}}{{"\n"}}' | awk -F ":" '{ print $2 }'
```

## Get EC2 instance ID given a kubernetes node ID 

`kubectl get node ip-10-80-103-53.eu-central-1.compute.internal -o yaml | grep providerID`

And then:

`aws ec2 terminate-instances --instance-ids ...`

## Change cronjob scheduling

```
$ kubectl get cronjob
NAME                   SCHEDULE        SUSPEND   ACTIVE   LAST SCHEDULE   AGE
toto                    0 5 */1 * *     False     0        3h53m           22d

$ kubectl patch cronjob toto -p '{"spec":{"schedule": "0 */1 */1 * *"}}'
cronjob.batch/toto patched

$ kubectl get cronjob
NAME                   SCHEDULE        SUSPEND   ACTIVE   LAST SCHEDULE   AGE
toto   0 */1 */1 * *   False     1        55m             22d
```

## Launch a job from an existing cronjob

`$ kubectl create job --from=cronjob/existing-cronjob my-job`
