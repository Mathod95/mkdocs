 # Kubernetes Master

  - Api server: Frontend for kubernetes, gestion users, cli, ...
  - etcd: key/value store use by kubernetes to managed date used by Kubernetes (If multiple node/master store all information to all nodes) lock
  - kubelet: agent that execture on each nodes of clusters, check that containers works properly on nodes
  - Container runtime: Docker
  - controller: noticing nodes, container and endpoints goes down, choose to brings up new container if neccessaru
  - scheduler: distributing works accross multiple nodes

# Master vs Workers nodes

  Workers run containers
  Container Runtime can be Docker, Rocket or CRI-O

  Master have the kube-apiserver Workers got the kubelet

  kubectl use to deploy and manage application on kubernetes cluster, get cluster informations, ...

Kubernetes à été conçu pour orchestrer Docker spécifiquement


# Object in YAML

Dans Kubernetes, un manifeste est un fichier de configuration (par exemple `pod.yaml`) qui contient la définition d’un ou plusieurs objets Kubernetes. Ces fichiers permettent de spécifier l’état souhaité du cluster, que Kubernetes appliquera automatiquement.
    
Une définition Kubernetes est la description complète d’un objet Kubernetes dans un fichier manifeste, incluant des champs tels que `apiVersion`, `kind`, `metadata`, ...

=== "accounts.yml"

    ```yaml title="pod.yaml" linenums="1"
    ---
    apiVersion: v1 #(1)!
    kind: Pod #(2)!
    metadata: #(3)!
      name: app1-pod #(4)!
      namespace: app1
      labels: #(5)!
        app: app1
        type: frontend
      annotations: #(6)!
        aws-account: "2389849082948"
        maintainer: "dev@mathod.io"
    spec: #(7)!
      containers: #(8)!
      - name: nginx-container
        image: nginx
    ```

    1.  The **apiVersion** field specifies the Kubernetes API version used to create the object.

        Keep in mind that other objects may require different versions, such as `apps/v1` for **Deployments** or `networking.k8s.io/v1` for **Ingress**
    2.  The **kind** field indicates the type of Kubernetes object to be created. For a Pod, it is defined as:
        ```
        kind: Pod
        ```
        
        For other object types, you might use values like `ReplicaSet`, `Deployment`, or `Service`.
    3.  In the **metadata** section, you provide key information about the Kubernetes object. 

        This includes the object's name, labels, ... which are key-value pairs that assist in organizing and filtering resources.
        !!! Warning 
            Do not include extra fields that Kubernetes doesn’t recognize under metadata, or it will reject your manifest.
            ??? info "Commonly used fields"
                - name
                - namespace
                - labels
                - annotations
                - uid
                - generation
                - creationTimestam
                - resourceVersion
                - finalizers
                - ownerReferences
                - managedFields
    4.  **TYPE STRING**

        !!! tips "Nommage clair et explicite"
            Appeler ton Pod app1-pod permet de savoir immédiatement qu’il s’agit d’un Pod et non:

            - d’un Deployment (app1-deploy),
            - d’un Service (app1-svc),
            - d’une ConfigMap (app1-cm),
            - ou d’un Namespace (app1).

            Cela aide beaucoup quand tu manipules plein de ressources avec `kubectl get all` ou dans une CI/CD.

            Si ton manifest est un `deployment`, `replicaSet` ou `job` il n'est pas nécéssaire d'ajouter la notion `<appName>-pod` il sera automatiquement générer

            !!! Example
                ```shell
                app1-deploy-79f8d9ccf5-v72rk
                ```
    5.  The labels field is a dictionary where you can define several key-value pairs.

        For instance, if you set labels such as app: myapp and type: frontend, it becomes easier to filter and manage your Pods later.
    6.  
    7.  The spec section describes the desired state of the object. 

        For a Pod, this primarily involves specifying the list of containers. Although a Pod can run multiple containers, this example focuses on a single container defined within an array for clarity. 
    8.  Each container configuration includes properties like the container's name and the Docker image used.