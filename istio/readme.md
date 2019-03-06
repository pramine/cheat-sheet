# Istio cheat sheet

* Scale Istio Pilot

`$ kubectl edit hpa istio-pilot`

change threshold (95 to 80 for example) in order to scale in the memory
