---
title: My Blog Post
date: 2025-09-29
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Tutorial
  - KinD
  - Cilium
  - eBPF
  - CNI
---

# Install Cilium in KinD

this is the text for my first blog post.
https://www.youtube.com/watch?v=pPEUhfTZswc

<!-- more -->

All the text here appears in the blog post

## Introduction

### Objectifs

### Prérequis

### Ma configuration

## Title1

Use `kind create cluster --config=kind-config.yaml` command to create the cluster

``` yaml title="kindClusterCilium.yaml"
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

### Title2
=== "Python"

    !!! note "Note for Python"
        This is a note inside the Python tab.

=== "JavaScript"

    !!! warning "Warning for JavaScript"
        This is a warning inside the JavaScript tab.
#### Title3

!!! note "Parent Admonition"

    This is the outer admonition content.

    === "Tab 1"

        !!! tip "Tip inside Tab 1"
            This is the first tab's inner admonition.

    === "Tab 2"

        !!! warning "Warning inside Tab 2"
            This is the second tab's inner admonition.

##### Title4

###### Title5

## Conclusion

### En rapport avec cet article

### Liens utile