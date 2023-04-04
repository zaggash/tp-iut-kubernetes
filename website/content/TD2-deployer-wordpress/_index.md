---
title: "TD - Deployer Wordpress"
weight: 5
pre: "<b>5. </b>"
draft: false
---

## Déployer Wordpress et MySQL avec du stockage et des Secrets

Nous allons suivre ce tutoriel pas à pas : https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

Il faut :
- copier les 2 fichiers et les appliquer
- vérifier que le stockage a bien fonctionné
- découvrir ce qui manque pour que cela fonctionne
- le créer à la main ou suivre le reste du tutoriel qui passe par l'outil Kustomize (attention, Kustomize ajoute un suffixe aux ressources qu'il créé)

On peut ensuite observer les différents objets créés, et optimiser le process avec un fichier `kustomization.yaml` plus complet.

- Entrez dans un des pods, et de l'intérieur, lisez le secret qui lui a été rendu accessible.


## Facultatif : la stack *Wordsmith*

Etudions et lançons ensemble cette application:

https://github.com/dockersamples/wordsmith
