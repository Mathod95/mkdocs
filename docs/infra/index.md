<figure markdown="1">
![](infra.svg)
</figure>

=== "Management"

    ```yaml
    apiVersion: kind.x-k8s.io/v1alpha4
    kind: Cluster
    name: management
    nodes:
    - role: control-plane
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
    ```

=== "Production"

    ```yaml
    apiVersion: kind.x-k8s.io/v1alpha4
    kind: Cluster
    name: production
    nodes:
    - role: control-plane
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
    ```

=== "Development"

    ```yaml
    apiVersion: kind.x-k8s.io/v1alpha4
    kind: Cluster
    name: development
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    ```

```shell
kind create cluster --config management.yaml
kind create cluster --config production.yaml
kind create cluster --config development.yaml
```

```shell
kubectx kind-management
```

```shell
export GIT_TOKEN=<TOKEN>
export GIT_REPO=https://gitea.mathod.fr/mathod/infra/bootstrap
```

```shell
kubens argocd
```

```shell
argocd login --core
Context 'kubernetes' updated

argocd cluster list
SERVER                          NAME        VERSION  STATUS      MESSAGE  PROJECT
https://kubernetes.default.svc  in-cluster  1.34     Successful

argocd cluster add
CURRENT  NAME             CLUSTER          SERVER
*        kind-management  kind-management  https://127.0.0.1:41469
         kind-production  kind-production  https://127.0.0.1:44821
         kind-staging     kind-development https://127.0.0.1:40971

argocd cluster add kind-production --project production
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-production` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:02+02:00"}
Cluster 'https://127.0.0.1:44821' added

❯ argocd cluster add kind-staging --project staging
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-staging` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:10+02:00"}
Cluster 'https://127.0.0.1:40971' added
```

```shell
argocd-autopilot project create production --dest-server 
argocd-autopilot project create development --dest-server 
argocd-autopilot project create management --dest-server in-cluster --project management --yes 
```

```shell
argocd-autopilot app create hello-world --app gitea.mathod.fr/mathod/infra/bootstrap/apps/examples -p production --wait-timeout 2m
argocd-autopilot app create hello-world --app gitea.mathod.fr/mathod/infra/bootstrap/apps/examples -p development --wait-timeout 2m

argocd-autopilot app create hello-world --installation-path bootstrap --repo https://gitea.mathod.fr/mathod/infra.git --app bootstrap/apps/examples -p production --wait-timeout 2m
argocd-autopilot app create hello-world --repo https://gitea.mathod.fr/mathod/infra.git --app bootstrap/apps/examples -p development --wait-timeout 2m
```

```shell
argocd cluster list
SERVER                          NAME              VERSION  STATUS      MESSAGE                                                  PROJECT
https://127.0.0.1:38013         kind-development           Unknown     Cluster has no applications and is not being monitored.  development
https://127.0.0.1:39635         kind-production            Unknown     Cluster has no applications and is not being monitored.
https://kubernetes.default.svc  in-cluster        1.34     Successful
```

```shell
argocd proj list
NAME         DESCRIPTION          DESTINATIONS  SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES  DESTINATION-SERVICE-ACCOUNTS
default                           *,*           *        */*                         <none>                        <none>          disabled            <none>
development  development project  *,*           *        */*                         <none>                        <none>          disabled            <none>
management   management project   *,*           *        */*                         <none>                        <none>          disabled            <none>
production   production project   *,*           *        */*                         <none>                        <none>          disabled            <none>
```

```shell
kind delete clusters management
kind delete clusters production
kind delete clusters development
```