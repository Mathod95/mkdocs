```ad-missing
Modifier les exemples de ce que peut faire Kyverno
```
## Introduction

[Kyverno](https://kyverno.io/) est un outil dédié à la sécurité et à la conformité des clusters Kubernetes. Il permet de créer et d'appliquer des _policies_ écrites en YAML de manière déclarative, proposant une approche plus flexible, native et facilite la sécurisation et le contrôle des ressources Kubernetes.

Par exemple, les policies peuvent servir à:
- Forcer la mise en place de labels afin d’harmoniser la gestion de vos ressources
- Restreindre la surface d’attaque en interdisant de monter des `hostPath`
- Empêcher un Pod d’utiliser des `hostPort`
- Mettre en place des valeurs par défaut pour les limitations de ressource

??? note "TLDR"
    Kyverno est un policy engine qui permet:  
    - Définir des policies en tant que ressources Kubernetes ;  
    - Valider, modifier ou générer des ressources à la volée via ces policies ; 
    - Bloquer des ressources non conformes grâce à un admission controller ;  
    - Journaliser les violations de policies dans des rapports. 

    Avantages:  
    - **Définir des policies de sécurité** pour interdire la création de ressources non sécurisées ;  
    - **Simplifier la vie des Ops** via des mutations de ressources à la volée ;  
    - **Possibilité de configurer les policies en mode audit** (sans blocage) ou enforce ;  
    - Une écriture de policy simple (par rapport à GateKeeper notamment)  

    Inconvénients:  
    - Difficile de créer des policies avec une logique très spécifique et/ou complexe ; 
    - Kyverno est un Single Point Of Failure. Certains connaissent [le côté sombre des adminssion controller](https://techblog.cisco  .com/blog/dark-side-of-kubernetes-admission-webhooks/) : si les pods Kyverno ne sont plus disponibles, plus aucune ressource Kubernetes ne peut se déployer sur le cluster. Je vous donnerai quelques astuces pour éviter tout problème dans la suite de l’article.


    2 types de policies 
    - **Policy**: S'applique à un namespace spécifique. Utile pour des configurations particulières à un environnement ou une applic  ation.
    - **ClusterPolicy**: S'applique à l'ensemble du cluster. Utilisée pour des règles de sécurité et de conformité globales.  

    3 types d'actions possible  
    - **Validation**: Kyverno permet de définir des règles pour vérifier que les configurations respectent certains standards avant   de créer ou modifier un objet. Par exemple, on peut interdire des privilèges trop élevés sur les Pods.
    - **Mutation**: Kyverno peut modifier des configurations par défaut. Par exemple, si un Pod n’a pas de valeur définie pour *labe  l*, Kyverno peut la générer automatiquement.
    - **Génération**: Il est possible de créer automatiquement des ressources selon certains déclencheurs, comme la création d’un _N  amespace_, pour garantir que chaque espace de noms ait une configuration de sécurité préconfigurée.

    2 modes d'utilisation 
    - **Audit**: Kyverno surveille les violations des _policies_ sans bloquer les opérations, permettant aux équipes de voir les inc  idents de non-conformité sans interrompre les déploiements.
    - **Strict**: Kyverno peut empêcher la création ou la mise à jour de ressources non conformes, offrant une approche de sécurité   proactive.


Il existe deux types de _policies_ dans Kyverno `Policy` et `ClusterPolicy`. Voici les différences entre les deux :
- **Policy**: Une _policy_ est une politique de validation, de mutation ou de génération qui s'applique à un _namespace_ spécifique dans un cluster Kubernetes. Les _policies_ sont utiles pour gérer des règles qui ne concernent qu'un environnement ou une application particulière, offrant ainsi une granularité dans l'application des règles de sécurité et de conformité. Par exemple, si vous avez plusieurs namespaces pour différents environnements (développement, test, production), vous pouvez créer des _policies_ spécifiques à chacun de ces namespaces.
- **ClusterPolicy**: Une **ClusterPolicy**, en revanche, est une politique qui s'applique à l'ensemble du cluster Kubernetes, sans restriction de namespace. Les _ClusterPolicies_ sont utilisées pour définir des règles de sécurité ou de conformité qui doivent être appliquées de manière globale à toutes les ressources du cluster. Cela peut inclure des exigences de sécurité fondamentales, comme l'utilisation de configurations spécifiques pour tous les Pods dans le cluster, indépendamment du namespace.

!!! info
    Ces deux ressources génèrent des `PolicyReports` (ou des `ClusterPolicyReports`)

C'est policy peuvent avoir 3 type d'actions !
- **Validation** : Kyverno permet de définir des règles pour vérifier que les configurations respectent certains standards avant de créer ou modifier un objet. Par exemple, on peut interdire des privilèges trop élevés sur les Pods.
- **Mutation** : Kyverno peut modifier des configurations par défaut. Par exemple, si un Pod n’a pas de valeur définie pour *label*, Kyverno peut la générer automatiquement.
- **Génération** : Il est possible de créer automatiquement des ressources selon certains déclencheurs, comme la création d’un _Namespace_, pour garantir que chaque espace de noms ait une configuration de sécurité préconfigurée.

Il existe 2 type de mode **Audit** et mode d’application **Strict** :
- En mode **audit**, Kyverno surveille les violations des _policies_ sans bloquer les opérations, permettant aux équipes de voir les incidents de non-conformité sans interrompre les déploiements.
- En mode **strict**, Kyverno peut empêcher la création ou la mise à jour de ressources non conformes, offrant une approche de sécurité proactive.

Kyverno passe à l’action à différents moments clés du cycle de vie des objets Kubernetes, en fonction du type de _policy_ et de l’action définie. Voici les principales situations où Kyverno intervient :
- À la création ou modification d’un objet Kubernetes
- Lors d’un événement déclencheur pour les policies de génération
- Pendant les audits périodiques

Kyverno intervient au moyen d’un Admission Controller dynamique qui traite des callbacks (webhooks d’admissions) de validation et de mutation en provenance de l’apiserver Kubernetes.

Les `Policies` Kyverno sont découpées en 3 parties:
- Les règles, ayant chacune un sélecteur et une action
- Le sélecteur de ressource de la règle, en inclusion ou en exclusion,
- L’action de la règle, qui peut être une validation, une mutation ou une génération de ressource
## Gérer ses Policies sur Kubernetes avec Kyverno

Déjà mentionné dans [notre article](https://particule.io/blog/kubernetes-psp-deprecated/) qui annonçait la dépréciation des PodSecurityPolicies et leur suppression à partir de la version 1.25, [Kyverno](https://kyverno.io/) s’annonce être un digne successeur à ces dernières.

Pour rappel, les PSP permettaient aux administrateurs de cluster de forcer l’application de bonnes pratiques en définissant un ensemble de règles qui étaient exécutées lors de la phase d’admission, au moyen d’un Admission Controller _compiled-in_.

Dans cet article, nous verrons comment mettre en place Kyverno, découvrir ses fonctionnalités et explorer ce que nous offre cette méthode de gestion des Policies.
### Objectifs
- 
### Prérequis
- Docker desktop installer
### Ma configuration
- Debian 12
- Kind v0.24.0
- [Docker desktop](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module) -  4.34.3

---
## Mise en place de Kyverno
## Installation

Commençons par mettre en place un environnement pour explorer les possibilités de Kyverno! En utilisant **kind** (présenté sur [notre blog](https://particule.io/blog/kubernetes-local/) !), configurons un cluster local:

``` bash hl_lines="1-3"
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

``` bash hl_lines="1"
kind version
kind v0.24.0 go1.22.6 linux/amd64
```

``` bash hl_lines="1"
kind create cluster --name kyverno
Creating cluster "kyverno" ...
 ✓ Ensuring node image (kindest/node:v1.31.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kyverno"
You can now use your cluster with:

kubectl cluster-info --context kind-kyverno

Thanks for using kind! 😊
```

Il existe différentes méthodes d’installation pour mettre en place Kyverno, utilisons le Chart Helm officiel pour bootstrapper notre cluster.

``` bash hl_lines="1 4 9"
helm repo add kyverno https://kyverno.github.io/kyverno/
"kyverno" has been added to your repositories

helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kyverno" chart repository
Update Complete. ⎈Happy Helming!⎈

helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
NAME: kyverno
LAST DEPLOYED: Sat Oct 19 19:41:37 2024
NAMESPACE: kyverno
STATUS: deployed
REVISION: 1
NOTES:
Chart version: 3.2.7
Kyverno version: v1.12.6

Thank you for installing kyverno! Your release is named kyverno.

The following components have been installed in your cluster:
- CRDs
- Admission controller
- Reports controller
- Cleanup controller
- Background controller


⚠️  WARNING: Setting the admission controller replica count below 2 means Kyverno is not running in high availability mode.

💡 Note: There is a trade-off when deciding which approach to take regarding Namespace exclusions. Please see the documentation at https://kyverno.io/docs/installation/#security-vs-operability to understand the risks.
```

!!! warning
    Vous remarquerez le **WARNING** lors de l'installation. Par défaut, Kyverno est déployé avec un seul réplica. En cas de crash, il devient impossible de déployer ou de modifier quoi que ce soit sur le cluster. La solution consiste à supprimer les **webhooks** de Kyverno pour débloquer la situation. Il est également recommandé d'augmenter ces limit en ressources (principalement RAM) et d’opter pour 3 réplicas (ou plus) afin de garantir plus de stabilité.


!!! note
    Il est probablement préférable d’exclure certains namespaces (kube-system, kyverno, …) du scope de Kyverno pour éviter de mauvaises surprises. Le blogpost _Thibault Lengagne_ de chez Padok en parle aussi (lien en bas de l’article).


``` bash hl_lines="1"
kubectl -n kyverno get all
NAME                                                           READY   STATUS      RESTARTS   AGE
pod/kyverno-admission-controller-84bc98464d-ppb8d              1/1     Running     0          57m
pod/kyverno-background-controller-86cdcb7fb6-prqst             1/1     Running     0          57m
pod/kyverno-cleanup-admission-reports-28822710-np9hw           0/1     Completed   0          8m57s
pod/kyverno-cleanup-cluster-admission-reports-28822710-fmj4g   0/1     Completed   0          8m57s
pod/kyverno-cleanup-cluster-ephemeral-reports-28822710-qphr5   0/1     Completed   0          8m57s
pod/kyverno-cleanup-controller-79cb7b5d87-qgl2l                1/1     Running     0          57m
pod/kyverno-cleanup-ephemeral-reports-28822710-cw5f7           0/1     Completed   0          8m57s
pod/kyverno-reports-controller-7c845bdf65-dkv4f                1/1     Running     0          57m

NAME                                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kyverno-background-controller-metrics   ClusterIP   10.96.93.154    <none>        8000/TCP   57m
service/kyverno-cleanup-controller              ClusterIP   10.96.91.249    <none>        443/TCP    57m
service/kyverno-cleanup-controller-metrics      ClusterIP   10.96.182.4     <none>        8000/TCP   57m
service/kyverno-reports-controller-metrics      ClusterIP   10.96.75.92     <none>        8000/TCP   57m
service/kyverno-svc                             ClusterIP   10.96.60.167    <none>        443/TCP    57m
service/kyverno-svc-metrics                     ClusterIP   10.96.251.154   <none>        8000/TCP   57m

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kyverno-admission-controller    1/1     1            1           57m
deployment.apps/kyverno-background-controller   1/1     1            1           57m
deployment.apps/kyverno-cleanup-controller      1/1     1            1           57m
deployment.apps/kyverno-reports-controller      1/1     1            1           57m

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/kyverno-admission-controller-84bc98464d    1         1         1       57m
replicaset.apps/kyverno-background-controller-86cdcb7fb6   1         1         1       57m
replicaset.apps/kyverno-cleanup-controller-79cb7b5d87      1         1         1       57m
replicaset.apps/kyverno-reports-controller-7c845bdf65      1         1         1       57m

NAME                                                      SCHEDULE       TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/kyverno-cleanup-admission-reports           */10 * * * *   <none>     False     0        8m57s           57m
cronjob.batch/kyverno-cleanup-cluster-admission-reports   */10 * * * *   <none>     False     0        8m57s           57m
cronjob.batch/kyverno-cleanup-cluster-ephemeral-reports   */10 * * * *   <none>     False     0        8m57s           57m
cronjob.batch/kyverno-cleanup-ephemeral-reports           */10 * * * *   <none>     False     0        8m57s           57m

NAME                                                           STATUS     COMPLETIONS   DURATION   AGE
job.batch/kyverno-cleanup-admission-reports-28822710           Complete   1/1           3s         8m57s
job.batch/kyverno-cleanup-cluster-admission-reports-28822710   Complete   1/1           3s         8m57s
job.batch/kyverno-cleanup-cluster-ephemeral-reports-28822710   Complete   1/1           3s         8m57s
job.batch/kyverno-cleanup-ephemeral-reports-28822710           Complete   1/1           3s         8m57s
```

Nous pouvons lister les CRDs installés par le chart Helm:

``` bash hl_lines="1 15"
kubectl api-resources --api-group kyverno.io
NAME                           SHORTNAMES   APIVERSION            NAMESPACED   KIND
admissionreports               admr         kyverno.io/v2         true         AdmissionReport
backgroundscanreports          bgscanr      kyverno.io/v2         true         BackgroundScanReport
cleanuppolicies                cleanpol     kyverno.io/v2         true         CleanupPolicy
clusteradmissionreports        cadmr        kyverno.io/v2         false        ClusterAdmissionReport
clusterbackgroundscanreports   cbgscanr     kyverno.io/v2         false        ClusterBackgroundScanReport
clustercleanuppolicies         ccleanpol    kyverno.io/v2         false        ClusterCleanupPolicy
clusterpolicies                cpol         kyverno.io/v1         false        ClusterPolicy
globalcontextentries           gctxentry    kyverno.io/v2alpha1   false        GlobalContextEntry
policies                       pol          kyverno.io/v1         true         Policy
policyexceptions               polex        kyverno.io/v2         true         PolicyException
updaterequests                 ur           kyverno.io/v2         true         UpdateRequest

kubectl api-resources --api-group wgpolicyk8s.io
NAME                   SHORTNAMES   APIVERSION                NAMESPACED   KIND
clusterpolicyreports   cpolr        wgpolicyk8s.io/v1alpha2   false        ClusterPolicyReport
policyreports          polr         wgpolicyk8s.io/v1alpha2   true         PolicyReport
```

Les règles du profile “default” sont définies via des `ClusterPolicies`:

``` bash hl_lines="1"
$ kubectl get cpol -n kyverno
NAME                             BACKGROUND   ACTION
disallow-add-capabilities        true         audit
disallow-host-namespaces         true         audit
disallow-host-path               true         audit
disallow-host-ports              true         audit
disallow-privileged-containers   true         audit
disallow-selinux                 true         audit
require-default-proc-mount       true         audit
restrict-apparmor-profiles       true         audit
restrict-sysctls                 true         audit
```

En effet, nous avons déjà un bon nombre de `Policies` ! Cependant, pas d’inquiétude. Nous pouvons observer qu’elles ont toutes été définies avec `background=true` et `action=audit`. Cette configuration indique à Kyverno que ces règles ne doivent pas être bloquantes. Toutes les 15 minutes, une analyse est lancée pour chaque règle sur les ressources présentes à cet instant au sein du Cluster et génèrent des ClusterPolicyReports.

### Exemples d’utilisation

#### Création d’une ClusterPolicy

Il est temps de mettre en place notre première `Policy` ! Définissons une règle simple qui vérifie la présence d’un label identifiant le nom de l’application à laquelle un `Pod` correspond.

``` yaml linenums="1"
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: enforce
  rules:
    - name: check-for-label-name
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "label `app.kubernetes.io/name` is required"
        pattern:
          metadata:
            labels:
              app.kubernetes.io/name: "?*"
```

La règle `check-for-label-name` permet de considérer comme prérequis le label `app.kubernetes.io/name` pour le déploiement des ressources de type `Pod`.

Pour résumer cette règle:

_Il est obligatoire pour tout `Pod` d’avoir un label `app.kubernetes.io/name` d’une longueur supérieure ou égale à 1 caractère._

Après avoir appliqué notre `ClusterPolicy`, nous obtenons la ressource suivante:

``` yaml linenums="1"
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
  annotations:
    pod-policies.kyverno.io/autogen-controllers: DaemonSet,Deployment,Job,StatefulSet,CronJob
spec:
  background: true
  rules:
    - match:
      resources:
        kinds:
          - Pod
      name: check-for-label-name
      validate:
        message: label `app.kubernetes.io/name` is required
      pattern:
        metadata:
          labels:
            app.kubernetes.io/name: ?*
    - match:
      resources:
        kinds:
          - DaemonSet
          - Deployment
          - Job
          - StatefulSet
      name: autogen-check-for-label-name
      validate:
        message: label `app.kubernetes.io/name` is required
        pattern:
          spec:
            template:
              metadata:
                labels:
                  app.kubernetes.io/name: ?*
    - match:
      resources:
        kinds:
          - CronJob
      name: autogen-cronjob-check-for-label-name
      validate:
        message: label `app.kubernetes.io/name` is required
        pattern:
          spec:
            jobTemplate:
              spec:
                template:
                  metadata:
                    labels:
                      app.kubernetes.io/name: ?*
      validationFailureAction: enforce
```

On retrouve notre règle “check-for-label-name”, mais deux autres règles ont été autogénérées afin d’appliquer le comportement que nous souhaitons aux autres ressources permettant de définir des pods.

Ces règles sont créées par les `autogen-controllers` ([link](https://kyverno.io/docs/writing-policies/autogen/)) référencés par l’annotation correspondante qui sert d’annotation par défaut. Définir manuellement cette annotation permet de modifier le comportement de génération automatique de règles.

Essayons de déployer une ressource qui enfreint la règle que nous venons de créer:

``` bash hl_lines="1 8"
$ kubectl run sample --image=busybox
Error from server: admission webhook "validate.kyverno.svc" denied the request:

resource Pod/default/sample was blocked due to the following policies

require-labels:
  check-for-labels: 'validation error: label `app.kubernetes.io/name` is required. Rule check-for-label-name failed at path /metadata/labels/app.kubernetes.io/name/'
$ kubectl run sample --image=busybox  --labels app.kubernetes.io/name=valid
pod/sample created
```

Notre `Policy` a une `validationFailureAction=enforce`, il nous est donc impossible de créer une ressource ne validant pas l’intégralité des règles définies!

Après avoir ajouté un label pour le nom de notre `Pod`, le webhook d’admission Kyverno valide la requête et la ressource est créée.

#### Le mode “audit”

Lors du déploiement du Chart Helm, un profil par défaut a été configuré en mode audit.

_We have installed the “default” profile of Pod Security Standards and set them in audit mode._

Ce mode de validation de `Policy` est _non-intrusif_ et permet de mettre progressivement en application les règles. Il n’empêche pas la création des ressources qui ne sont pas conformes à la `Policy` mais va historiser les infractions dans des `PolicyReport` (`polr`) si la ressource est _namespaced_ (comme un `Deployment` ou un `ConfigMap`) dans le namespace de la ressource, ou des `ClusterPolicyReports` (`cpolr`) s’il s’agit de ressource non _namespaced_ (un `Namespace`)

Lors de la création d’un `Deployment`, le template du `Pod` enfreint une `ClusterPolicy`. L’infraction sera reportée dans une `PolicyReport` dans le namespace du `Pod`.

Lors de la création d’une `IngressClass`, l’absence de label enfreint une autre `ClusterPolicy`. L’infraction sera reportée dans une `ClusterPolicyReport`.

Comme mentionné précédemment, le mode audit va périodiquement (toutes les 15 minutes) analyser les ressources et générer des reports.

Configurons une `Policy` en mode audit ainsi qu’un `Pod` de test:

``` yaml linenums="1"
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-tier-on-pod
spec:
  validationFailureAction: audit
  background: true
  rules:
    - name: check-for-tier-on-pod
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "if set, label `app.kubernetes.io/tier` must be any of ['frontend', 'backend', 'internal']"
        pattern:
          metadata:
            labels:
              =(app.kubernetes.io/tier): "frontend | backend | internal"
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app.kubernetes.io/tier: uknown
    app.kubernetes.io/name: example-pod
  name: bad-tier
spec:
  containers:
    - image: nginx:latest
      name: example-tier
```

_Si le label `app.kubernetes.io/tier` existe, il doit correspondre à l’une des valeurs frontend, backend ou internal._

Après quelques instants, nous pouvons observer un avertissement venant de l’admission-controller:

``` bash hl_lines="1 8"
$ kubectl describe pod bad-tier  | grep -A 5 Events
Events:
  Type     Reason           Age   From                  Message
  ----     ------           ----  ----                  -------
  Warning  PolicyViolation  8s   admission-controller  Rule(s) 'check-for-tier-on-pod' of policy 'require-tier-on-pod' failed to apply on the resource
  Normal   Scheduled        8s   default-scheduler     Successfully assigned default/bad-tier to kyverno-control-plane
  Normal   Pulling          7s   kubelet               Pulling image "nginx:latest"
$ kubectl get polr
NAME              PASS   FAIL   WARN   ERROR   SKIP   AGE
polr-ns-default   1      1      0      0       0      30m
```

Nous pouvons obtenir un _report_ complet en effectuant un _describe_ sur le `PolicyReport` du `Namespace` correspondant.

``` yaml linenums="1"
---
apiVersion: wgpolicyk8s.io/v1alpha1
kind: PolicyReport
metadata:
  name: polr-ns-default
  namespace: default
results:
  - message: 'validation error: if set, label `app.kubernetes.io/tier` must be any of
    [''frontend'', ''backend'', ''internal'']. Rule check-for-tier-on-pod failed at
    path /metadata/labels/app.kubernetes.io/tier/'
    policy: require-tier-on-pod
    resources:
      - apiVersion: v1
        kind: Pod
        name: bad-tier
        namespace: default
        uid: 6e743322-5b2a-4ad7-bc79-eca437ab82db
    rule: check-for-tier-on-pod
    scored: true
    status: fail
  - category: Pod Security Standards (Default)
    message: validation rule 'host-namespaces' passed.
    policy: disallow-host-namespaces
    resources:
      - apiVersion: v1
        kind: Pod
        name: sample
        namespace: default
        uid: 19dc0c5b-3960-4178-8bb0-6cd61a23c089
    rule: host-namespaces
    scored: true
    status: pass
summary:
  error: 0
  fail: 1
  pass: 1
  skip: 0
  warn: 0
```

Les `PolicyReports` permettent d’avoir une vue d’ensemble des infractions locales au namespace dans lequel elles résident.

Chaque report inclut un récapitulatif de l’application des règles avec le nombre d’infraction, de validation, d’avertissements, …

### Comment implémenter Kyverno

#### Lors de la mise en place d’un Cluster

Afin de partir sur de bonnes bases, vous pouvez intégrer Kyverno à votre Cluster Kubernetes après sa création en surchargeant [les valeurs par défaut](https://github.com/kyverno/kyverno/blob/main/charts/kyverno/values.yaml) du Chart Helm.

Une version plus sécurisée du profile “[default](https://kyverno.io/policies/pod-security/default/)” est disponible via le profile “[restricted](https://kyverno.io/policies/pod-security/restricted/)” et peut être accompagnée d’un `validationFailureAction=enforce` afin de garantir l’intégrité et la sécurité du cluster.

#### Dans un Cluster existant

Il est totalement possible d’ajouter Kyverno à une configuration en utilisant le même procédé. Attention cependant à ne pas enforce les Policies que vous mettez en place de façon à ne pas cause d’interruption.

Une transition vers des ressources conformes aux règles que vous souhaitez définir pourrait s’effectuer de en utilisant la méthode suivante:

- Définition des règles à implémenter via Kyverno en mode audit
- Observer les infractions via les PolicyReports et les ClusterPolicyReports
- Ajout d’une étape de validation dans les CI/CD via [Kyverno CLI](https://kyverno.io/docs/kyverno-cli/)
- Passage en mode “enforce” lorsque les ressources sont conformes à la Policy


## Conclusion

La mise en place d’une solution de gestion de Policy est une étape importante concernant la sécurisation des clusters Kubernetes.

Kyverno réussit à répondre à cette problématique d’une manière élégante et peut se révéler être un excellent remplaçant aux PSP. La dimension _Kubernetes Native_ rend la courbe d’apprentissage très douce, ne nécessitant pas d’apprendre de nouvelle syntaxe.

Dans cet article, nous avons couvert les similarités au niveau des PSP. Cependant, Kyverno propose des fonctionnalités additionnelles, telle que la [mutation](https://kyverno.io/docs/writing-policies/mutate/) de ressources permettant de rajouter des valeurs par défauts, ou encore la [génération](https://kyverno.io/docs/writing-policies/validate/) de ressources qui offre la possibilité de répliquer des ressources à travers différents namespaces ou d’automatiser certaines opérations.

[**Theo “Bob” Massard**](https://www.linkedin.com/in/tbobm/), Cloud Native Engineer

### En rapport avec cet article
### Liens utile
