---
title: "TD - Installation et Configuration"
weight: 3
pre: "<b>3. </b>"
draft: false
---

## Découverte de Kubernetes

### Installer le client CLI `kubectl`

kubectl est le point d'entré universel pour contrôler tous les type de cluster kubernetes. 
C'est un client en ligne de commande qui communique en REST avec l'API d'un cluster.

Nous allons explorer kubectl au fur et à mesure des TPs. Cependant à noter que :

- `kubectl` peut gérer plusieurs clusters/configurations et switcher entre ces configurations
- `kubectl` n'est pas le seul outils pour intéragir avec les cluster, mais il est l'outil officiel.
- Il est recommender d'utiliser une version qui correspond à la version de votre cluster Kubernetes.

La méthode d'installation importe peu. 
Vous pouvez suivre la methode officiel sur le [site de Kubernetes](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)

- Faites `kubectl version` pour afficher la version du client kubectl.  

##### Bash completion

Pour permettre à `kubectl` de compléter le nom des commandes et ressources avec `<Tab>` il est utile d'installer l'autocomplétion pour Bash :

```bash
sudo apt install bash-completion

source <(kubectl completion bash)

echo "source <(kubectl completion bash)" >> ${HOME}/.bashrc
```

**Vous pouvez désormais appuyer sur `<Tab>` pour compléter vos commandes `kubectl`, c'est très utile !**

### Installer Kubernetes

Nous allons créer un cluster kubernetes en utilisant l’installeur RKE (Rancher Kubernetes Engine).  
RKE est un installeur qui utilise docker pour contenir les différents composant de Kubernetes.  
Il est donc nécessaire de disposer de machines où docker est installé et fonctionnel.  
  
#### Preparation  

Nous allons utiliser [cet article](https://www.suse.com/c/rancher_blog/an-introduction-to-rancher-kubernetes-engine-rke/) comme support.  

##### Verifier les machines

Leur donner un nom identifiable.  
Verifier l'installation de Docker. 
Avoir un utilisateur en commun, cet utilisateur doit faire partie du groupe Docker.

##### Definir les accès SSH

Créez une paire de clefs ssh sans passphrase sur le master (commande ssh-keygen) et placez les dans `.ssh/id_rsa` et `.ssh/id_rsa.pub`. 
Ne mettez pas de mot de passe.
  
Ajoutez la clef au fichier des clefs autorisées (.ssh/authorized_keys).  
Attention de conserver les clefs déjà présentes (celles qui vous permettent de vous connecter à ces serveurs).

Vérifier le fonctionnement en testant la connexion depuis l'une des machines du cluster sur les autres.

#### Installation avec RKE

RKE va nous simplifier la tache et va installer Kubernetes en fonction de nos paramètres spécifiés dans le fichier `cluster.yaml`

Récupérer la derniere **version stable** de RKE depuis [le depot GitHub](https://github.com/rancher/rke/releases)
Le fichier qui est fourni est un simple fichier exécutable, vous pouvez changer ses droits et l'utilisez le avec la command:  
```
./rke config
```  

Cette commande est un script interactif qui pose des questions pour créer le fichier de configuration du cluster.  

***Commencez par créer un cluster sur vos 3 machines, la première devant assurer le rôle de `controle-plane` et celui de `etcd`, les deux autres seront `worker`.***

Il est possible de modifier le fichier après sa création. 
Vous trouverez dans [la documentation officielle](https://rke.docs.rancher.com/example-yamls), un exemple court et un exemple avec toutes les options !


Attention, dans la plupart des cas, il faut utiliser les réponses par défaut.
Cette commande va créer le fichier de configuration du cluster `cluster.yml` qu'il est possible d'éditer à la main si vous avez fait une petite erreur lors du script.

L'installation se lance simplement avec la commande:
```
./rke up --config cluster.yaml
```

Si tout se passe bien, la commande se termine au bout de quelques minutes par
```
[...]

INFO[0327] Finished building Kubernetes cluster successfully
```

### Explorons notre cluster k8s

Notre cluster k8s est plein d'objets divers, organisés entre eux de façon dynamique pour décrire des applications, tâches de calcul, services et droits d'accès. La première étape consiste à explorer un peu le cluster :

- Listez les nodes pour récupérer le nom de vos noeuds (`kubectl get nodes`) puis affichez ses caractéristiques avec `kubectl describe node/<nom-du-noeud>`.

La commande `get` est générique et peut être utilisée pour récupérer la liste de tous les types de ressources.

De même, la commande `describe` peut s'appliquer à tout objet k8s. On doit cependant préfixer le nom de l'objet par son type (ex : `node/<nom-du-noeud>` ou `nodes <nom-du-noeud>`) car k8s ne peut pas deviner ce que l'on cherche quand plusieurs ressources ont le même nom.

- Pour afficher tous les types de ressources à la fois que l'on utilise : `kubectl get all`

```
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1   <none>        443/TCP   2m34s
```

Il semble qu'il n'y a qu'une ressource dans notre cluster. Il s'agit du service d'API Kubernetes, pour qu'on puisse communiquer avec le cluster.

En réalité il y en a généralement d'autres cachés dans les autres `namespaces`.  
En effet les éléments internes de Kubernetes tournent eux-mêmes sous forme de services et de daemons Kubernetes.  
Les *namespaces* sont des groupes qui servent à isoler les ressources de façon logique et en termes de droits (avec le *Role-Based Access Control* (RBAC) de Kubernetes).

Pour vérifier cela on peut :

- Afficher les namespaces : `kubectl get namespaces`

Un cluster Kubernetes a généralement un namespace appelé `default` dans lequel les commandes sont lancées et les ressources créées si on ne précise rien.  
Il a également aussi un namespace `kube-system` dans lequel résident les processus et ressources système de k8s.  
Pour préciser le namespace on peut rajouter l'argument `-n` à la plupart des commandes k8s.

- Pour lister les ressources liées au `kubectl get all -n kube-system`.

- Ou encore : `kubectl get all --all-namespaces` (peut être abrégé en `kubectl get all -A`) qui permet d'afficher le contenu de tous les namespaces en même temps.

- Pour avoir des informations sur un namespace : `kubectl describe namespace/kube-system`

### Déployer une application en CLI

Nous allons maintenant déployer une première application conteneurisée.  
Le déploiement est un peu plus complexe qu'avec Docker, en particulier car il est séparé en plusieurs objets et plus configurable.

- Pour créer un déploiement en ligne de commande (par opposition au mode déclaratif que nous verrons plus loin).
On peut lancer par exemple: 
```
kubectl create deployment rancher-demo --image=monachus/rancher-demo
```

Cette commande crée un objet de type `deployment`. Nous pourvons étudier ce deployment avec la commande `kubectl describe deployment/rancher-demo`.

- Notez la liste des événements sur ce déploiement en bas de la description.
- De la même façon que dans la partie précédente, listez les `pods` avec `kubectl`. Combien y en a-t-il ?

- Agrandissons ce déploiement avec `kubectl scale deployment rancher-demo --replicas=5`
- `kubectl describe deployment/rancher-demo` permet de constater que le service est bien passé à 5 replicas.
  - Observez à nouveau la liste des évènements, le scaling y est enregistré...
  - Listez les pods pour constater

A ce stade impossible d'afficher l'application : le déploiement n'est pas encore accessible de l'extérieur du cluster. Pour régler cela nous devons l'exposer grace à un service :
```
kubectl expose deployment rancher-demo --type=NodePort --port=8080 --name=rancher-demo-service
```

- Affichons la liste des services pour voir le résultat: `kubectl get services`

Un service permet de créer un point d'accès unique exposant notre déploiement. Ici nous utilisons le type Nodeport car nous voulons que le service soit accessible de l'extérieur par l'intermédiaire d'un forwarding de port.

- Sauriez-vous expliquer ce que l'app fait ?
- Pour le comprendre ou le confirmer, diminuez le nombre de réplicats à l'aide de la commande utilisée précédement pour passer à 10 réplicas. Qu se passe-t-il ?


Une autre méthode pour accéder à un service (quel que soit sont type) en mode développement est de forwarder le traffic par l'intermédiaire de kubectl (et des composants kube-proxy installés sur chaque noeuds du cluster).

- Pour cela on peut par exemple lancer: `kubectl port-forward svc/rancher-demo-service 8080:8080 --address 127.0.0.1`
- Vous pouvez désormais accéder à votre app via via kubectl sur: `http://localhost:8080`. Quelle différence avec l'exposition précédente ?

=> Un seul conteneur s'affiche. En effet `kubectl port-forward` sert à créer une connexion de developpement/debug qui pointe toujours vers le même pod en arrière plan.

Pour exposer cette application en production sur un véritable cluster, nous devrions plutôt avoir recours à service de type un LoadBalancer. Mais RKE ne propose pas par défaut de loadbalancer. Nous y reviendrons dans le cours sur les objets kubernetes.

#### Simplifier les lignes de commande k8s

- Pour gagner du temps on dans les commandes Kubernetes on peut définir un alias: `alias kc='kubectl'` (à mettre dans votre `.bash_profile` en faisant `echo "alias kc='kubectl'" >> ~/.bash_profile`, puis en faisant `source ~/.bash_profile`).
- Vous pouvez ensuite remplacer `kubectl` par `kc` dans les commandes.

- Également pour gagner du temps en ligne de commande, la plupart des mots-clés de type Kubernetes peuvent être abrégés :
  - `services` devient `svc`
  - `deployments` devient `deploy`
  - etc.

La liste complète : <https://blog.heptio.com/kubectl-resource-short-names-heptioprotip-c8eff9fb7202>

- Essayez d'afficher les serviceaccounts (users) et les namespaces avec une commande courte.
