```ad-missing
- Revoir le texte
- https://docs.crossplane.io/knowledge-base/guides/import-existing-resources/#import-resources-manually
```

Avant on utilisait Terraform et on en a créé des ressources ! Maintenant qu'on joue avec Crossplane on aimerait bien importer les ressources précédemment managé par Terraform pour les intégrer dans Kubernetes et les manager via Crossplane
# Introduction

Si vous disposez de ressources déjà provisionnées chez AWS, vous pouvez les importer en tant que ressources gérées par Crossplane mais pas si vite ! Dans un premier temps nous allons d'abord apprendre à observer des ressources puis par la suite les managé via Crossplane
# Objectifs

- Apprendre à observer des ressources AWS avec Crossplane
- Apprendre à importer des ressources AWS avec Crossplane manuellement
- Apprendre à importer des ressources AWS avec Crossplane automatiquement

---
# Observer une ressources

Crossplane peut découvrir et observer des ressources existantes sur AWS en faisant correspondre l'annotation `crossplane.io/external-name` avec le nom de la ressource créer sur AWS et nous ajoutons également l'option `managementPolicies: ["Observe"]`.

Le champ `managementPolicies: ["Observe"]` permet à Crossplane d'importer la ressource mais en aucun cas n'appliquera de modification ou supprimera cette dernière.

Pour observer une ressource externe existante depuis votre provider, créez une nouvelle ressource gérée avec l'annotation `crossplane.io/external-name`. Définissez la valeur de l'annotation comme étant le nom de la ressource chez votre Provider.

Ci dessous un exemple, pour importer un VPC existant, créez une nouvelle ressource gérée et utilisez son ID dans l'annotation.

## Appliquer la managementPolicies: Observe

Créer une nouvelle ressource gérée correspondant à l'apiVersion et au type de la ressource à importer et ajouter managementPolicies : ["Observe"] à la spécification

Par exemple, pour importer un VPC de AWS, créer une nouvelle ressource avec l'ensemble managementPolicies : ["Observe"].

```yaml title:importVPC.yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
spec:
  managementPolicies: ["Observe"]
```

## Ajouter l'annotation external-name

Ajouter l'annotation `crossplane.io/external-name` pour la ressource. Ce nom doit correspondre au nom à l'intérieur du fournisseur.

Par exemple, pour une base de données GCP nommée my-external-database, appliquez l'annotation `crossplane.io/external-name` avec la valeur my-external-database.

importVpc.yaml

```yaml title:importVPC.yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
metadata:
  annotations:
    crossplane.io/external-name: vpc-029aa16a171ccb018
spec:
  managementPolicies: ["Observe"]
```

## Créer un nom d'objet Kubernetes

Par exemple, nommez l'objet Kubernetes imported-vpc

importVpc.yaml

```yaml title:importVPC.yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
metadata:
  annotations:
    crossplane.io/external-name: vpc-029aa16a171ccb018
  name: imported-vpc
spec:
  managementPolicies: ["Observe"]
```

## Configurer les champs required du spec.forProvider

```yaml title:importVPC.yaml 
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
metadata:
  annotations:
    crossplane.io/external-name: vpc-029aa16a171ccb018
  name: imported-vpc
spec:
  forProvider:
    region: eu-west-1
  managementPolicies: ["Observe"]
```

---

Crossplane peut découvrir et observer des ressources existantes sur AWS en faisant correspondre l'annotation `crossplane.io/external-name` avec le nom de la ressource créer sur AWS. Nous appliquerons également l'option `managementPolicies: ["Observe"]` pour nous permettre dans un premier temps d'avoir une ressources en mode readOnly

Le champ `managementPolicies: ["Observe"]` permet à Crossplane d'importer la ressource mais en aucun cas n'appliquera de modification ou supprimera cette dernière.

Pour observer une ressource externe existante depuis votre provider, créez une nouvelle ressource gérée avec l'annotation `crossplane.io/external-name`. Définissez la valeur de l'annotation comme étant le nom de la ressource chez votre Provider.

Par exemple, pour importer un VPC existant, créez une nouvelle ressource gérée et utilisez son ID dans l'annotation.

```yaml title:importVPC.yaml
apiVersion: ec2.aws.upbound.io/v1beta1
kind: VPC
metadata:
  annotations:
    crossplane.io/external-name: vpc-029aa16a171ccb018
  name: imported-vpc
spec:
  forProvider:
    region: eu-west-1
  managementPolicies: ["Observe"]
```

Le champ `metadata.name` peut être tout ce que vous voulez. Par exemple, imported-vpc. Il représenteras le nom de l'objet Kubernetes