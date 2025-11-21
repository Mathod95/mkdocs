```shell title="Merge kubeconfig"
KUBECONFIG=$HOME/mgmt/kubeconfig:$HOME/prod/kubeconfig kubectl config view --flatten > ~/.kube/config
```