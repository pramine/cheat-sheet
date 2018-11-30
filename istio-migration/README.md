# Istio migration

With istio 0.8 and 1.0 comes several modifications to do in your side, new configuration resources to control traffic routing into, within, and out of the mesh.

Goodbye Ingress, RouteRule, DestinationPolicy and EgressRule and welcome to Gateway, VirtualService, DestinationRule and ServiceEntry.

New traffic management APIs, aka v1alpha3: not backward compatible and will require manual conversion from the old API.

## Changed resources

Shortly:

   * Ingress -> Gateway
   * RouteRule -> VirtualService
   * DestinationPolicy -> DestinationRule
   * EgressRule -> ServiceEntry

![Istio v1alpha3 routing schema](https://istio.io/blog/2018/v1alpha3-routing/virtualservices-destrules.svg)

## General

### Gateway

Configures a load balancer for HTTP/TCP traffic, regardless of where it will be running. Any number of gateways can exist within the mesh and multiple different gateway implementations can co-exist.

For example, the following simple Gateway configures a load balancer to allow external https traffic for host google.com into the mesh:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: google-gateway
spec:
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - google.com
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
```

### VirtualService

To configure the corresponding routes, a VirtualService must be defined for the same host and bound to the Gateway using the gateways field in the configuration:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: google
spec:
  hosts:
    - google.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings
```

### DestinationRule

A DestinationRule configures the set of policies to be applied while forwarding traffic to a service. They are intended to be authored by service owners, describing the circuit breakers, load balancer settings, TLS settings, etc..

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

### ServiceEntry

Everything you could previously configure using an EgressRule can just as easily be done with a ServiceEntry. For example, access to a simple external service from inside the mesh can be enabled using a configuration something like this:

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: foo-ext
spec:
  hosts:
  - foo.com
  ports:
  - number: 80
    name: http
    protocol: HTTP
```

## Tools

`$ istioctl experimental convert-ingress`