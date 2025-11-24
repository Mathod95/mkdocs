---
hide:
  - tags
tags:
  - CheatSheet
  - Kubernetes
todo:
---

# Kubernetes cheat sheet
Kubernetes cheatsheet featuring all the necessary commands required to manage a Kubernetes cluster.

```shell
k         "kubectl"                                          # The kubectl command
kca       "kubectl --all-namespaces"                         # The kubectl command targeting all namespaces
kaf       "kubectl apply --filename"                         # Apply a YML file
keti      "kubectl exec -ti"                                 # Drop into an interactive terminal on a container

# Context management
kcuc      "kubectl config use-context"                       # Set the current-context in a kubeconfig file
kcsc      "kubectl config set-context"                       # Set a context entry in kubeconfig
kcdc      "kubectl config delete-context"                    # Delete the specified context from the kubeconfig
kccc      "kubectl config current-context"                   # Display the current-context
kcgc      "kubectl config get-contexts"                      # List of contexts available

# General aliases
kdel      "kubectl delete"                                   # Delete resources by filenames, stdin, resources and names, or by resources and label selector
kdelf     "kubectl delete --filename"                        # Delete a pod using the type and name specified in --filename argument

# Pod management
kgp       "kubectl get pods"                                 # List all pods in ps output format
kgpl      "kubectl get pods --selector"                      # Get pods by label. Example: "kgpl "app=myapp" -n myns"
kgpn      "kubectl get pods --namespace="                    # Get pods by namespace. Example: "kgpn kube-system"
kgpsl     "kubectl get pods --show-labels"                   # List all pods in ps output format with labels
kgpw      "kubectl get pods --watch"                         # After listing/getting the requested object, watch for changes
kgpwide   "kubectl get pods --output wide"                   # Output in plain-text format with any additional information. For pods, the node name is included
kep       "kubectl edit pods"                                # Edit pods from the default editor
kdp       "kubectl describe pods"                            # Describe all pods
kdelp     "kubectl delete pods"                              # Delete all pods matching passed arguments

# Service management
kgs       "kubectl get svc"                                  # List all services in ps output format
kgsw      "kubectl get svc --watch"                          # After listing all services, watch for changes
kgswide   "kubectl get svc --output wide"                    # After listing all services, output in plain-text format with any additional information
kes       "kubectl edit svc"                                 # Edit services(svc) from the default editor
kds       "kubectl describe svc"                             # Describe all services in detail
kdels     "kubectl delete svc"                               # Delete all services matching passed argument

# Ingress management
kgi       "kubectl get ingress"                              # List ingress resources in ps output format
kei       "kubectl edit ingress"                             # Edit ingress resource from the default editor
kdi       "kubectl describe ingress"                         # Describe ingress resource in detail
kdeli     "kubectl delete ingress"                           # Delete ingress resources matching passed argument

# Namespace management
kgns      "kubectl get namespaces"                           # List the current namespaces in a cluster
kcn       "kubectl config set-context --current --namespace" # Change current namespace
kens      "kubectl edit namespace"                           # Edit namespace resource from the default editor
kdns      "kubectl describe namespace"                       # Describe namespace resource in detail
kdelns    "kubectl delete namespace"                         # Delete the namespace. WARNING! This deletes everything in the namespace

# ConfigMap management
kgcm      "kubectl get configmaps"                           # List the configmaps in ps output format
kecm      "kubectl edit configmap"                           # Edit configmap resource from the default editor
kdcm      "kubectl describe configmap"                       # Describe configmap resource in detail
kdelcm    "kubectl delete configmap"                         # Delete the configmap

# Secret management
kgsec     "kubectl get secret"                               # Get secret for decoding
kdsec     "kubectl describe secret"                          # Describe secret resource in detail
kdelsec   "kubectl delete secret"                            # Delete the secret

# Deployment management
kgd       "kubectl get deployment"                           # Get the deployment
kgdw      "kubectl get deployment --watch"                   # After getting the deployment, watch for changes
kgdwide   "kubectl get deployment --output wide"             # After getting the deployment, output in plain-text format with any additional information
ked       "kubectl edit deployment"                          # Edit deployment resource from the default editor
kdd       "kubectl describe deployment"                      # Describe deployment resource in detail
kdeld     "kubectl delete deployment"                        # Delete the deployment
ksd       "kubectl scale deployment"                         # Scale a deployment
krsd      "kubectl rollout status deployment"                # Check the rollout status of a deployment
kres      "kubectl set env $@ REFRESHED_AT=..."              # Recreate all pods in deployment with zero-downtime

# Rollout management
kgrs      "kubectl get replicaset"                           # List all ReplicaSets "rs" created by the deployment
kdrs      "kubectl describe replicaset"                      # Describe ReplicaSet in detail
kers      "kubectl edit replicaset"                          # Edit ReplicaSet from the default editor
krh       "kubectl rollout history"                          # Check the revisions of this deployment
kru       "kubectl rollout undo"                             # Rollback to the previous revision

# Port forwarding
kpf       "kubectl port-forward"                             # Forward one or more local ports to a pod

# Tools for accessing all information
kga       "kubectl get all"                                  # List all resources in ps format
kgaa      "kubectl get all --all-namespaces"                 # List the requested object(s) across all namespaces

# Logs
kl        "kubectl logs"                                     # Print the logs for a container or resource
klf       "kubectl logs --filename"                          # Stream the logs for a container or resource (follow)

# File copy
kcp       "kubectl cp"                                       # Copy files and directories to and from containers

# Node management
kgno      "kubectl get nodes"                                # List the nodes in ps output format
kgnosl    "kubectl get nodes --show-labels"                  # List the nodes in ps output format with labels
keno      "kubectl edit node"                                # Edit nodes resource from the default editor
kdno      "kubectl describe node"                            # Describe node resource in detail
kdelno    "kubectl delete node"                              # Delete the node

# Persistent Volume Claim management
kgpvc     "kubectl get pvc"                                  # List all PVCs
kgpvcw    "kubectl get pvc --watch"                          # After listing/getting the requested object, watch for changes
kepvc     "kubectl edit pvc"                                 # Edit pvcs from the default editor
kdpvc     "kubectl describe pvc"                             # Describe all pvcs
kdelpvc   "kubectl delete pvc"                               # Delete all pvcs matching passed arguments

# StatefulSets management
kgss      "kubectl get statefulset"                          # List the statefulsets in ps format
kgssw     "kubectl get statefulset --watch"                  # After getting the list of statefulsets, watch for changes
kgsswide  "kubectl get statefulset --output wide"            # After getting the statefulsets, output in plain-text format with any additional information
kess      "kubectl edit statefulset"                         # Edit statefulset resource from the default editor
kdss      "kubectl describe statefulset"                     # Describe statefulset resource in detail
kdelss    "kubectl delete statefulset"                       # Delete the statefulset
ksss      "kubectl scale statefulset"                        # Scale a statefulset
krsss     "kubectl rollout status statefulset"               # Check the rollout status of a deployment

# Service Accounts management
kdsa      "kubectl describe sa"                              # Describe a service account in details
kdelsa    "kubectl delete sa"                                # Delete the service account

# DaemonSet management
kgds      "kubectl get daemonset"                            # List all DaemonSets in ps output format
kgdsw     "kubectl get daemonset --watch"                    # After listing all DaemonSets, watch for changes
keds      "kubectl edit daemonset"                           # Edit DaemonSets from the default editor
kdds      "kubectl describe daemonset"                       # Describe all DaemonSets in detail
kdelds    "kubectl delete daemonset"                         # Delete all DaemonSets matching passed argument

# CronJob management
kgcj      "kubectl get cronjob"                              # List all CronJobs in ps output format
kecj      "kubectl edit cronjob"                             # Edit CronJob from the default editor
kdcj      "kubectl describe cronjob"                         # Describe a CronJob in details
kdelcj    "kubectl delete cronjob"                           # Delete the CronJob

# Job management
kgj       "kubectl get job"                                  # List all Job in ps output format
kej       "kubectl edit job"                                 # Edit a Job in details
kdj       "kubectl describe job"                             # Describe the Job
kdelj     "kubectl delete job"                               # Delete the Job
```

??? Tips "ABBR"

    ``` shell
    abbr "k"="kubectl"
    abbr "kca"="kubectl --all-namespaces"
    abbr "kaf"="kubectl apply -f"
    abbr "keti"="kubectl exec -ti"
    # Context
    abbr "kctx"="kubectx"
    abbr "kcuc"="kubectl config use-context"
    abbr "kcsc"="kubectl config set-context"
    abbr "kcdc"="kubectl config delete-context"
    abbr "kccc"="kubectl config current-context"
    abbr "kcgc"="kubectl config get-contexts"
    # General aliases
    abbr "kdel"="kubectl delete"
    abbr "kdelf"="kubectl delete -f"
    # Pod management
    abbr "kgp"="kubectl get pods"
    abbr "kgpl"="kubectl get pods -l"
    abbr "kgpn"="kubectl get pods -n"
    abbr "kgpsl"="kubectl get pods --show-labels"
    abbr "kgpw"="kubectl get pods --watch"
    abbr "kgpwide"="kubectl get pods -o wide"
    abbr "kep"="kubectl edit pods"
    abbr "kdp"="kubectl describe pods"
    abbr "kdelp"="kubectl delete pods"
    # Service management
    abbr "kgs"="kubectl get svc"
    abbr "kgsw"="kubectl get svc --watch"
    abbr "kgswide"="kubectl get svc -o wide"
    abbr "kes"="kubectl edit svc"
    abbr "kds"="kubectl describe svc"
    abbr "kdels"="kubectl delete svc"
    # Ingress management
    abbr "kgi"="kubectl get ingress"
    abbr "kei"="kubectl edit ingress"
    abbr "kdi"="kubectl describe ingress"
    abbr "kdeli"="kubectl delete ingress"
    # Namespace management
    abbr "kns"="kubens"
    abbr "kgns"="kubectl get namespaces"
    abbr "kcn"="kubectl config set-context --current --namespace"
    abbr "kens"="kubectl edit namespace"
    abbr "kdns"="kubectl describe namespace"
    abbr "kdelns"="kubectl delete namespace"
    # ConfigMap management
    abbr "kgcm"="kubectl get configmaps"
    abbr "kecm"="kubectl edit configmap"
    abbr "kdcm"="kubectl describe configmap"
    abbr "kdelcm"="kubectl delete configmap"
    # Secret management
    abbr "kgsec"="kubectl get secret"
    abbr "kdsec"="kubectl describe secret"
    abbr "kdelsec"="kubectl delete secret"
    # Deployment management
    abbr "kgd"="kubectl get deployment"
    abbr "kgdw"="kubectl get deployment --watch"
    abbr "kgdwide"="kubectl get deployment -o wide"
    abbr "ked"="kubectl edit deployment"
    abbr "kdd"="kubectl describe deployment"
    abbr "kdeld"="kubectl delete deployment"
    abbr "ksd"="kubectl scale deployment"
    abbr "krsd"="kubectl rollout status deployment"
    abbr "kres"="kubectl set env $@ REFRESHED_AT=..."
    # Rollout management
    abbr "kgrs"="kubectl get replicaset"
    abbr "kdrs"="kubectl describe replicaset"
    abbr "kers"="kubectl edit replicaset"
    abbr "krh"="kubectl rollout history"
    abbr "kru"="kubectl rollout undo"
    # Port forwarding
    abbr "kpf"="kubectl port-forward"
    # Tools for accessing all information
    abbr "kga"="kubectl get all"
    abbr "kgaa"="kubectl get all --all-namespaces"
    # Logs
    abbr "kl"="kubectl logs"
    abbr "klf"="kubectl logs -f"
    # File copy
    abbr "kcp"="kubectl cp"
    # Node management
    abbr "kgno"="kubectl get nodes"
    abbr "kgnosl"="kubectl get nodes --show-labels"
    abbr "keno"="kubectl edit node"
    abbr "kdno"="kubectl describe node"
    abbr "kdelno"="kubectl delete node"
    # Persistent Volume Claim management
    abbr "kgpvc"="kubectl get pvc"
    abbr "kgpvcw"="kubectl get pvc --watch"
    abbr "kepvc"="kubectl edit pvc"
    abbr "kdpvc"="kubectl describe pvc"
    abbr "kdelpvc"="kubectl delete pvc"
    # StatefulSets management
    abbr "kgss"="kubectl get statefulset"
    abbr "kgssw"="kubectl get statefulset --watch"
    abbr "kgsswide"="kubectl get statefulset -o wide"
    abbr "kess"="kubectl edit statefulset"
    abbr "kdss"="kubectl describe statefulset"
    abbr "kdelss"="kubectl delete statefulset"
    abbr "ksss"="kubectl scale statefulset"
    abbr "krsss"="kubectl rollout status statefulset"
    # Service Accounts management
    abbr "kdsa"="kubectl describe sa"
    abbr "kdelsa"="kubectl delete sa"
    # DaemonSet management
    abbr "kgds"="kubectl get daemonset"
    abbr "kgdsw"="kubectl get daemonset --watch"
    abbr "keds"="kubectl edit daemonset"
    abbr "kdds"="kubectl describe daemonset"
    abbr "kdelds"="kubectl delete daemonset"
    # CronJob management
    abbr "kgcj"="kubectl get cronjob"
    abbr "kecj"="kubectl edit cronjob"
    abbr "kdcj"="kubectl describe cronjob"
    abbr "kdelcj"="kubectl delete cronjob"
    # Job management
    abbr "kgj"="kubectl get job"
    abbr "kej"="kubectl edit job"
    abbr "kdj"="kubectl describe job"
    abbr "kdelj"="kubectl delete job"
    ```