```shell
helm repo add crossplane-stable https://charts.crossplane.io/stable
"crossplane-stable" has been added to your repositories
```

```shell
helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "crossplane-stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

```shell
helm search repo crossplane
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
crossplane-stable/crossplane    2.1.1           2.1.1           Crossplane is an open source Kubernetes add-on ...
```


```shell
helm install crossplane --namespace crossplane-system --create-namespace --version 2.1.1 crossplane-stable/crossplane
NAME: crossplane
LAST DEPLOYED: Sun Nov 16 19:22:29 2025
NAMESPACE: crossplane-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Release: crossplane

Chart Name: crossplane
Chart Description: Crossplane is an open source Kubernetes add-on that enables platform teams to assemble infrastructure from multiple vendors, and expose higher level self-service APIs for application teams to consume.
Chart Version: 2.1.1
Chart Application Version: 2.1.1

Kube Version: v1.34.0
```

```shell
kubectl get pods -n crossplane-system
NAME                                       READY   STATUS    RESTARTS   AGE
crossplane-6896f6fbff-sn2x8                1/1     Running   0          98s
crossplane-rbac-manager-65f6d66c76-76ffs   1/1     Running   0          98s
```

```shell
kubectl get crds | grep crossplane
clusterusages.protection.crossplane.io                          2025-11-16T18:22:34Z
compositeresourcedefinitions.apiextensions.crossplane.io        2025-11-16T18:22:34Z
compositionrevisions.apiextensions.crossplane.io                2025-11-16T18:22:34Z
compositions.apiextensions.crossplane.io                        2025-11-16T18:22:34Z
configurationrevisions.pkg.crossplane.io                        2025-11-16T18:22:34Z
configurations.pkg.crossplane.io                                2025-11-16T18:22:34Z
cronoperations.ops.crossplane.io                                2025-11-16T18:22:34Z
deploymentruntimeconfigs.pkg.crossplane.io                      2025-11-16T18:22:34Z
environmentconfigs.apiextensions.crossplane.io                  2025-11-16T18:22:34Z
functionrevisions.pkg.crossplane.io                             2025-11-16T18:22:34Z
functions.pkg.crossplane.io                                     2025-11-16T18:22:34Z
imageconfigs.pkg.crossplane.io                                  2025-11-16T18:22:34Z
locks.pkg.crossplane.io                                         2025-11-16T18:22:34Z
managedresourceactivationpolicies.apiextensions.crossplane.io   2025-11-16T18:22:34Z
managedresourcedefinitions.apiextensions.crossplane.io          2025-11-16T18:22:34Z
operations.ops.crossplane.io                                    2025-11-16T18:22:34Z
providerrevisions.pkg.crossplane.io                             2025-11-16T18:22:34Z
providers.pkg.crossplane.io                                     2025-11-16T18:22:34Z
usages.apiextensions.crossplane.io                              2025-11-16T18:22:34Z
usages.protection.crossplane.io                                 2025-11-16T18:22:34Z
watchoperations.ops.crossplane.io                               2025-11-16T18:22:34Z
```

```bash
cat << 'EOF' > provider-aws-s3.yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: upbound-provider-aws-s3
spec:
  package: xpkg.upbound.io/upbound/provider-aws-s3:v2.2.0
EOF
```

```shell
kubectl apply -f provider-aws-s3.yaml
provider.pkg.crossplane.io/upbound-provider-aws-s3 created
```

```shell
kubectl get providers
NAME                          INSTALLED   HEALTHY   PACKAGE                                              AGE
upbound-provider-aws-s3       True        True      xpkg.upbound.io/upbound/provider-aws-s3:v2.2.0       2m30s
upbound-provider-family-aws   True        True      xpkg.upbound.io/upbound/provider-family-aws:v2.2.0   2m25s
```

When you install an aws-provider it install will also check if the family provider is installed

```shell
kubectl get pods -n crossplane-system
NAME                                                        READY   STATUS    RESTARTS   AGE
crossplane-6896f6fbff-sn2x8                                 1/1     Running   0          12m
crossplane-rbac-manager-65f6d66c76-76ffs                    1/1     Running   0          12m
upbound-provider-aws-s3-dd62e217befa-85fffd899-qjpzj        1/1     Running   0          4m23s
upbound-provider-family-aws-0e8a4558ea96-6bff6ccb65-f2x5s   1/1     Running   0          4m24s
```

```shell
kubectl get crds | grep aws.upbound.io
bucketaccelerateconfigurations.s3.aws.upbound.io                2025-11-16T18:30:27Z
bucketacls.s3.aws.upbound.io                                    2025-11-16T18:30:28Z
bucketanalyticsconfigurations.s3.aws.upbound.io                 2025-11-16T18:30:28Z
bucketcorsconfigurations.s3.aws.upbound.io                      2025-11-16T18:30:26Z
bucketintelligenttieringconfigurations.s3.aws.upbound.io        2025-11-16T18:30:26Z
bucketinventories.s3.aws.upbound.io                             2025-11-16T18:30:27Z
bucketlifecycleconfigurations.s3.aws.upbound.io                 2025-11-16T18:30:28Z
bucketloggings.s3.aws.upbound.io                                2025-11-16T18:30:27Z
bucketmetrics.s3.aws.upbound.io                                 2025-11-16T18:30:26Z
bucketnotifications.s3.aws.upbound.io                           2025-11-16T18:30:28Z
bucketobjectlockconfigurations.s3.aws.upbound.io                2025-11-16T18:30:26Z
bucketobjects.s3.aws.upbound.io                                 2025-11-16T18:30:28Z
bucketownershipcontrols.s3.aws.upbound.io                       2025-11-16T18:30:27Z
bucketpolicies.s3.aws.upbound.io                                2025-11-16T18:30:28Z
bucketpublicaccessblocks.s3.aws.upbound.io                      2025-11-16T18:30:26Z
bucketreplicationconfigurations.s3.aws.upbound.io               2025-11-16T18:30:27Z
bucketrequestpaymentconfigurations.s3.aws.upbound.io            2025-11-16T18:30:28Z
buckets.s3.aws.upbound.io                                       2025-11-16T18:30:26Z
bucketserversideencryptionconfigurations.s3.aws.upbound.io      2025-11-16T18:30:26Z
bucketversionings.s3.aws.upbound.io                             2025-11-16T18:30:26Z
bucketwebsiteconfigurations.s3.aws.upbound.io                   2025-11-16T18:30:29Z
directorybuckets.s3.aws.upbound.io                              2025-11-16T18:30:29Z
objectcopies.s3.aws.upbound.io                                  2025-11-16T18:30:26Z
objects.s3.aws.upbound.io                                       2025-11-16T18:30:28Z
providerconfigs.aws.upbound.io                                  2025-11-16T18:30:23Z
providerconfigusages.aws.upbound.io                             2025-11-16T18:30:23Z
```

Create an user with admin privilege

```shell
[default]
aws_access_key_id = XXXXXXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

```shell
kubectl create secret generic aws-secret -n crossplane-system --from-file=creds=./aws-credentials.txt
```

https://www.youtube.com/watch?v=jw8mMslpqOI
use namespaced scopped

https://docs.crossplane.io/v2.1/guides/upgrade-to-crossplane-v2/
https://docs.crossplane.io/v2.1/whats-new/

```yaml
---
apiVersion: aws.m.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
        namespace: crossplane-system
        name: aws-secret
        key: creds
```

```yaml
---
apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket
metadata:
  name: app1-bucket-mathod
  namespace: crossplane-system
spec:
  forProvider:
    region: eu-west-3
      tags:
        company: mathod
        project: app1
        environment: production
    providerConfigRef:
      name: default
```

## Komoplane

``` shell
helm repo add komodorio https://helm-charts.komodor.io
"komodorio" has been added to your repositories
```
```shell
helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "komodorio" chart repository
...Successfully got an update from the "crossplane-stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```
```shell
helm upgrade --install komoplane komodorio/komoplane
Release "komoplane" does not exist. Installing it now.
NAME: komoplane
LAST DEPLOYED: Mon Nov 17 15:42:15 2025
NAMESPACE: crossplane-system
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing Komoplane.
Application can be accessed:
  * Within your cluster, at the following DNS name at port 8090:

    komoplane.crossplane-system.svc.cluster.local

  * From outside the cluster, run these commands in the same shell:

    export POD_NAME=$(kubectl get pods --namespace crossplane-system -l "app.kubernetes.io/name=komoplane,app.kubernetes.io/instance=komoplane" -o jsonpath="{.items[0].metadata.name}")
    export CONTAINER_PORT=$(kubectl get pod --namespace crossplane-system $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
    echo "Visit http://127.0.0.1:8090 to use your application"
    kubectl --namespace crossplane-system port-forward $POD_NAME 8090:$CONTAINER_PORT

Visit our repo at:
https://github.com/komodorio/komoplane
```


helm upgrade --install komoplane komodorio/komoplane --namespace komoplane --create-namespace --set service.type=NodePort --set service.nodePort=30080 --set extraArgs="--port=8090"