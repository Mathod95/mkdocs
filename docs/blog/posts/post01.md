---
title: My Blog Post
date: 2025-10-03
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Tutorial
  - KinD
  - Cilium
  - eBPF
  - CNI
sources:
  - https://medium.com/@nahelou.j/play-with-cilium-native-routing-in-kind-cluster-5a9e586a81ca
  - https://medium.com/@charled.breteche/kind-cluster-with-cilium-and-no-kube-proxy-c6f4d84b5a9d
---

# Install Cilium in KinD

this is the text for my first blog post.

<!-- more -->
## Introduction



### Objectifs

Ce guide vous explique comment déployer un cluster Kind (Kubernetes dans Docker) configuré pour :

- désactiver le CNI par défaut de Kind (kindnet)
- désactiver kube-proxy
- utiliser Cilium comme CNI avec un remplacement strict du proxy
- utiliser le mode de routage natif / routage direct
- activer en option :
    - le masquage basé sur eBPF
    - les annonces L2
    - l’accélération XDP
    - les pools d’adresses IP pour le load-balancer (LB)

L’objectif est de simuler localement une configuration de type production, avec moins de dépendances à iptables/ipvs, de meilleures performances, et une couche réseau plus proche de celle des clusters réels.

### Prérequis

### Ma configuration
- kind v0.29.0 go1.24.3 linux/amd64

---

## Configuration de KinD

``` yaml title="kind-config.yaml" linenums="1"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  disableDefaultCNI: true
  kubeProxyMode: none
```
Running the command below will spin up a cluster without kindnet and will not install kube-proxy:
```bash hl_lines="1"
kind create cluster --config=kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.33.1) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:
kubectl cluster-info --context kind-kind
Have a nice day! 👋
```
??? success "OUTPUT"
    Lorsque le cluster démarre suite à `kind create cluster --config=kind-config.yaml`, de nombreux pods resteront en `Pending` et les nœuds pourront apparaître comme `NotReady`, car aucun CNI n’est encore présent.
    ``` bash hl_lines="1"
    kubectl get pods --all-namespaces
    NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
    kube-system          coredns-674b8bbfcf-9gvrc                     0/1     Pending   0          61m
    kube-system          coredns-674b8bbfcf-rlx56                     0/1     Pending   0          61m
    kube-system          etcd-kind-control-plane                      1/1     Running   0          61m
    kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          61m
    kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          61m
    kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          61m
    local-path-storage   local-path-provisioner-7dc846544d-hztkz      0/1     Pending   0          61m
    ```
    ``` bash hl_lines="1"
    kubectl get nodes
    NAME                 STATUS     ROLES           AGE   VERSION
    kind-control-plane   NotReady   control-plane   24h   v1.33.1
    kind-worker          NotReady   <none>          24h   v1.33.1
    kind-worker2         NotReady   <none>          24h   v1.33.1
    ```

## Installation de Cilium

``` bash
helm repo add cilium https://helm.cilium.io
"cilium" has been added to your repositories
```
``` bash
helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cilium" chart repository
Update Complete. ⎈Happy Helming!⎈
```
``` bash
helm show values cilium/cilium > cilium-values.yaml
```

``` bash
helm upgrade --install --namespace kube-system --repo https://helm.cilium.io cilium cilium --values - <<EOF
kubeProxyReplacement: strict
hostServices:
  enabled: false
externalIPs:
  enabled: true
nodePort:
  enabled: true
hostPort:
  enabled: true
image:
  pullPolicy: IfNotPresent
ipam:
  mode: kubernetes
hubble:
  enabled: true
  relay:
    enabled: true
  ui:
    enabled: true
EOF
```

``` yaml title="cilium-config.yaml" linenums="1"
kubeProxyReplacement: strict
k8sServiceHost: kind-control-plane
k8sServicePort: 6443

ipam:
  mode: kubernetes

routingMode: native
ipv4NativeRoutingCIDR: 10.244.0.0/16
autoDirectNodeRoutes: true

bpf:
  masquerade: true
ipMasqAgent:
  enabled: true
  config:
    nonMasqueradeCIDRs:
      - 10.244.0.0/8

l2announcements:
  enabled: true

loadBalancer:
  acceleration: native
  mode: hybrid

hubble:
  relay:
    enabled: true
  ui:
    enabled: true
```

```
$ helm install -n kube-system cilium cilium/cilium -f cilium.yaml
```
``` yaml
cluster:
  name: kind-kind

ipv4:
  enabled: true
ipv6:
  enabled: false

enableIPv4Masquerade: true
```

---

## Conclusion

### En rapport avec cet article

### Liens utile