# Kubernetes

Basically, if you have a deployment, and you update the container to a new version, this is what happens in parallel (paraphrased from a comment on that thread):

* Kubelet sends a SIGTERM to the pod
* A controller removes the pod from the endpoints
* kube-proxy removes the Pod from virtual LBs

