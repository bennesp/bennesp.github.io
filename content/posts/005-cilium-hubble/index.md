---
title: "Explore Cilium Hubble 🛰️"
date: 2024-12-18T18:45:12+01:00
cover:
  image: imgs/cover.png
summary: Some neat features of Hubble, leveraged by Cilium
tags:
  - kubernetes
  - cilium
  - hubble
  - networking
  - security
  - observability
---

Cilium is now installed [after these steps]({{< relref "004-k3s-cilium" >}}), but it doesn't mean
much as of now. To see the tip of the benefits iceberg, we need to explore Hubble.

Hubble is the observability layer of Cilium. It allows you to monitor the network traffic flowing
through your nodes and pods.

## 1. Install Hubble

Install it with:

```sh
helm upgrade cilium cilium/cilium \
   --namespace kube-system \
   --reuse-values \
   --set hubble.relay.enabled=true \
   --set hubble.ui.enabled=true
```

And wait for Cilium to be ready with `cilium status --wait`:

{{< figure src="imgs/cilium-wait.png" align=center caption="cilium status --wait after some minutes while Hubble is being installed" >}}

## 2. Explore Hubble UI

Hubble is the observability layer of Cilium.
Hubble UI is the web interface to explore the data collected by Hubble.

Enable it with:

```sh
cilium hubble ui
```

Open the URL in your browser and select the namespace you want to explore.

{{< figure src="imgs/cover.png" align=center caption="Hubble UI service graph, in a namespace of choice" >}}

## 3. Explore Hubble CLI

The main potential of Hubble can be glimpsed by using its CLI.

First of all let's start the port-forwarding to the Hubble Relay:

```sh
kubectl port-forward -n kube-system deploy/hubble-relay 4245:grpc
```

Then, we can use the `hubble observe` command to see the traffic flowing through the network (`-f` follows the traffic):

```sh
hubble observe --namespace traefik -f
```

## 4. Explore L7 visibility

Cilium and Hubble can provide Layer 7 visibility, meaning that you can see the HTTP or DNS requests
flowing through your network in real-time, and even filter them.

To see L7 flows, we need to enable L7 visibility for a specific namespace or pod. However, before
enabling L7 visibility, we need to have a CiliumNetworkPolicy in place.

This is a simple tailored policy for one of my application pods.

```yaml {linenos=inline,hl_lines=["20-21","25-26","36-37","47-49"]}
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: immich-l7-visibility
  namespace: immich
spec:
  endpointSelector:
    matchLabels:
      k8s:io.kubernetes.pod.namespace: immich
      app.kubernetes.io/name: server
  ingress:
    # Allow all traffic coming from otel namespace to metrics ports
    - fromEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: otel
      toPorts:
        - ports:
            - port: "8081"
              protocol: TCP
          rules:
            http: [{}]
        - ports:
            - port: "8082"
              protocol: TCP
          rules:
            http: [{}]
    # Allow all traffic coming from the internet to http port
    - fromEndpoints:
        - matchLabels:
            app.kubernetes.io/name: traefik
            io.kubernetes.pod.namespace: traefik
      toPorts:
        - ports:
            - port: "2283"
              protocol: TCP
          rules:
            http: [{}]
  egress:
    # Allow all traffic going to the DNS server
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: kube-system
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP
          rules:
            dns:
              - matchPattern: "*"
    # Allow all traffic going to Redis
    - toEndpoints:
        - matchLabels:
            app.kubernetes.io/name: redis
            io.kubernetes.pod.namespace: immich
      toPorts:
        - ports:
            - port: "6379"
              protocol: TCP
    # Allow all traffic going to PostgreSQL
    - toEndpoints:
        - matchLabels:
            app.kubernetes.io/name: postgresql
            io.kubernetes.pod.namespace: immich
      toPorts:
        - ports:
            - port: "5432"
              protocol: TCP
```

**Note**: the highlighted lines are the ones that enable L7 visibility for the specified ports.

In this case I enabled L7 visibility for 4 ports: 8081, 8082, 2283, and 53.

After that, we can simply observe L7 traffic with:

```sh
hubble observe --namespace immich -f -t l7
```

and we can see all the flows with their details, including HTTP URLs, status codes and DNS queries!

{{< figure src="imgs/l7-flows.png" align=center caption="Hubble CLI showing L7 visibility for a specific namespace" >}}
