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

### Prérequis

### Ma configuration

## Title1

!!! note "Global Admonition"

    !!! info "First Code Block"
        ``` yaml title="kind-config.yaml"
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

    ??? info "OUTPUT"
        === "Create cluster"
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

        === "Pods Pending"
            Une fois la commande `kind create cluster --config=kind-config.yaml` lancée pour créer le cluster, on remarque que certains pods restent dans l’état « Pending ». Ce phénomène est dû à l’absence de plugin CNI dans le cluster pour gérer les adresses IP internes (pods et services). Procédons au déploiement de Cilium dans notre cluster kind pour résoudre ce problème.

            ``` bash hl_lines="3 4 9"
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

### Title2

#### Title3

##### Title4

###### Title5

## Conclusion

### En rapport avec cet article

### Liens utile