---
title: "Migrate K3s to Cilium"
date: 2024-12-17T13:05:44+01:00
cover:
  image: imgs/cover.png
summary: A step-by-step guide to migrate your K3s cluster from default CNI to Cilium
tags:
  - kubernetes
  - k3s
  - cilium
  - cni
  - networking
---

Suppose you have a running k3s cluster and would like to explore Cilium.
You don't want to start from scratch because you would have to redeploy all your applications and possibly lose data.

However, migrating the k3s CNI doesn't seem to be officially supported (see [this discussion](https://github.com/k3s-io/k3s/discussions/8101)).

So here you are, copying and pasting from the internet and hoping it works.

These are the (minimum) steps I followed to migrate my k3s cluster to Cilium during a coffee break from work ☕️

# Steps
## Prerequisites
- A running K3s cluster
- Administrative access to the cluster nodes
- The Cilium CLI tool installed
- A backup of critical data (recommended)

## 🔌 1. Disable previous CNI

Edit file `/etc/rancher/k3s/config.yaml` (create it if it doesn't exist)

```yaml {linenos=inline}
flannel-backend: none
disable-network-policy: true
```

And simply restart k3s to apply the changes.

```sh
sudo systemctl restart k3s
```

## 🔌 2. Remove old flannel interface
I am not sure why this is not done automatically, but it is a mandatory step, otherwise Cilium will
not be able to start failing with the impossibility of creating a vxlan interface.
```sh
ip link delete flannel.1
```

## 📦 3. Install Cilium
```sh
cilium install --set ipam.operator.clusterPoolIPv4PodCIDRList="10.42.0.0/16"
cilium status --wait
```

After some seconds/minutes, Cilium will be ready.

{{<figure src="imgs/cover.png" align=center caption="Cilium after some minutes when it is ready">}}

## ⚠️ 4. Restart all the pods

Since Cilium is now managing the network, but pods are still using the old CNI,
you **must** to restart all your pods.

This is one of the reason I believe that many providers do not support this migration.

### 4.1. Restart all the pods safely (more or less)

```sh
kubectl get ns -o name | sed 's|namespace/||' | xargs -I{} kubectl rollout restart deployment -n {}
kubectl get ns -o name | sed 's|namespace/||' | xargs -I{} kubectl rollout restart statefulset -n {}
kubectl get ns -o name | sed 's|namespace/||' | xargs -I{} kubectl rollout restart daemonset -n {}
```

### 4.2. Simpler way of restarting all the pods

However, if you can accept the risk of doing so (and of course seconds/minutes of downtime),
you can simply delete all the pods, causing them to be restarted with the new CNI.

```sh
kubectl delete pod --all --all-namespaces
```
