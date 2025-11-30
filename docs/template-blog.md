---
title: "Titre de ton article"
description: "Courte description pour l’aperçu et le SEO"
date: 2025-02-10
tags:
  - tutoriel
  - mkdocs
  - guide
categories:
  - Documentation
author: "Mathias FELIX"
thumbnail: images/mkdocs-thumb.png
other: https://www.youtube.com/watch?v=pPEUhfTZswc
---

## Introduction
Contexte + intérêt du sujet.  
Phrase contenant ton mot-clé principal.

### Objectifs
  - découvrir comment organiser un article dans MkDocs Material
  - utiliser une structure claire et hiérarchique
  - faciliter la lecture grâce à des sections adaptées

### Prérequis
  - connaître les bases du Markdown
  - utiliser MkDocs et le thème Material
  - savoir lancer `mkdocs serve`

### Ma configuration
  - **OS :** Ubuntu 22.04  
  - **MkDocs :** 1.6.x  
  - **Theme Material :** 9.x  
  - **Plugins :** blog, search  

---

## Title1

### Title2

#### Title3

##### Title4

###### Title5

---

## Conclusion

### En rapport avec cet article

### Liens utile



----------- HELPER -----------
```yaml
apiVersion: v2
name: crossplane
description: A Helm chart for Crossplane
type: application
version: "0.1.0" #(1)!
appVersion: "2.1.1" #(2)!

dependencies:
  - name: crossplane
    version: "2.1.1"
    repository: "https://charts.crossplane.io/stable"
```

  1.  **REQUIRED**
  
      **version**: La version de ton manifest Chart.yaml.  
      
  2.  **OPTIONAL**
  
      **appVersion**: La version de l'application que tu déploie.