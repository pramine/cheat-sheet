# Istio cheat sheet

* Scale Istio Pilot

`$ kubectl edit hpa istio-pilot -n istio-system`

change threshold (95 to 80 for example) in order to scale in the memory.

Example:

```
spec:
  maxReplicas: 5
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1beta1
    kind: Deployment
    name: istio-pilot
  targetCPUUtilizationPercentage: 80
```

* I have deployment but my pods can't be scheduled, why?

```
$ kubectl get deployment
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
presto-coordinator   1         1         1            0           38m
presto-worker        2         2         2            0           38m

$ kubectl get po
No resources found.

$ kubectl get deployment presto-coordinator -o yaml
apiVersion: extensions/v1beta1
kind: Deployment
...
status:
  conditions:
  - lastTransitionTime: "2019-05-22T07:14:25Z"
    lastUpdateTime: "2019-05-22T07:14:25Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2019-05-22T07:14:26Z"
    lastUpdateTime: "2019-05-22T07:14:26Z"
    message: 'Internal error occurred: failed calling admission webhook "sidecar-injector.istio.io":
      Post https://istio-sidecar-injector.istio-system.svc:443/inject?timeout=30s:
      dial tcp 100.68.225.18:443: connect: connection refused'
```

Ok so we've got an issue with sidecar-injector.

Check if the pod is running:

`$ kubectl get po -n istio-system`

Ok, the pod is not running, a node have been drained and the pod was running in this drained node, so run it again.

You need to force to scale down and up in order to have your pods.

```
$ kubectl scale deploy presto-coordinator --replicas=0
deployment.extensions/presto-coordinator scaled

$ kubectl scale deploy presto-coordinator --replicas=1
deployment.extensions/presto-coordinator scaled

$ kubectl scale deploy presto-worker --replicas=0
deployment.extensions/presto-worker scaled

$ kubectl scale deploy presto-worker --replicas=2
deployment.extensions/presto-worker scaled

$ kubectl get po
NAME                                  READY   STATUS              RESTARTS   AGE
presto-coordinator-5c7547854b-nk97p   1/1     Running             0          7m55s
presto-worker-68f64f9bb4-74dd7        0/1     ContainerCreating   0          7m39s
presto-worker-68f64f9bb4-pnvdj        1/1     Running             0          7m39s
```

Great!
