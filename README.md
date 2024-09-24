<p align="center">
    <img src="https://github.com/user-attachments/assets/3ba5a526-c617-49c7-8165-30c3f3505d5c" width="300" alt="TSP logo">
</p>

# CSC8567 - Architectures distribuées et applications web
## Projet Final Kubernetes

## Auteurs

- Timothée Mathubert (timothee.mathubert@telecom-sudparis.eu)
- Gatien Roujanski (gatien.roujanski@telecom-sudparis.eu)
- Arthur Jovart (arthur.jovart@telecom-sudparis.eu)
- (Inspirations du cours NET4255 de Vincent Gauthier & Gatien Roujanski)

## Démarrage des projets

Pour cette partie du cours, vous aurez besoin d'installer le programme `kubectl`.
1. Veuillez suivre les instructions disponibles sur ce [tuto d'installation de `kubectl`](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/).
2. Venez ensuite voir un professeur : donnez la composition de votre groupe. Un namespace vous sera alors créé sur le cluster du cours (`groupe-X` où X est le numéro de votre groupe). Un compte vous sera aussi créé sur le cluster.
3. Une fois votre compte créé, [connectez-vous au site de gestion du cluster](https://kube.luxbulb.org) avec vos identifiants, et créez vous un mot de passe personnel.
4. Allez ensuite dans la rubrique "Cluster Management". 
5. Sélectionnez le cluster "csc8567", et cliquez sur "Download KubeConfig".
6. Déplacez le fichier téléchargé à l'adresse `~/.kube/config` (`config` n'est pas un répertoire, c'est bien le fichier de configuration que vous avez téléchargé : il faut le renommer).
7. Essayez la commande dans un terminal (en remplaçant le "X" par votre numéro de groupe) :
```
kubectl cluster-info -n groupe-X
```
*Pour info, la notion `-n groupe-X` permet de préciser que la commande est exécutée dans l'espace de noms "groupe-X". Sans la mention de ce dernier, elle serait exécutée dans le namespace "default", auquel vous n'avez pas accès. Probablement une info utile pour la suite !*

Ceci devrait vous afficher une sortie :
```
Kubernetes control plane is running at https://kube.luxbulb.org/k8s/clusters/local

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
Si vous avez une erreur, allez voir un professeur pour résoudre le problème.
8. Une fois que la dernière commande à fonctionné, vous êtes fin prêts pour démarrer le projet !

## Modalités de rendu & soutenance

Vous êtes répartis en groupes de 2 ou 3 personnes, et vous partagez l'espace de noms associé à votre groupe.
Le rendu final sera commun au membres du groupe : une seule personne du groupe le rendra sur Moodle, et précisera les noms des membres du groupe.
La note se basera sur votre avancée dans les **Défis** listés ci-après.

Chaque Défi contient deux sections. Il faut les compléter pour réussir chaque Défi.
- Contenu : les tâches à réaliser.
- Questions : les questions auxquelles il faut répondre.

Votre rendu sera une archive `zip` ou `tar.gz` contenant, **pour chaque Défi** :
- Dans le cas où seules des commandes étaient utiles à la réalisation du Défi, un fichier contenant ligne par ligne les commandes que vous avez exécutées pour réussir le Contenu.
- Les fichiers de configuration que vous avez appliqués pour réussir un Défi. *Dans le cas où des fichiers de configurations ont été utilisés, il n'est pas nécessaire de repréciser les commandes que vous avez exécutées pour les appliquer.*
- Un schéma d'infrastructure représentant tous les composants réseau participant au fonctionnement du service. *Référez-vous aux les présentations et n'hésitez pas à poser des questions si vous avez des doutes sur certains points !*
- Les réponses aux questions de chaque Défi.

Pour la soutenance, vous devez laisser sur le cluster du cours et dans votre espace de noms tous les déploiements que vous avez fait. **Ne les supprimez pas après avoir rendu l'archive !!**

Le jour de la soutenance :
- **Venez 15mn avant l'horaire de passage, avec l'ordinateur prêt à présenter.**
- Annoncez le dernier Défi réalisé.

On espère que cette partie va vous plaire, et vous donner des idées plus claire sur la puissance de ce super outil qu'est Kubernetes !

## Premiers pas sur Kubernetes (Défi 1)

### Contenu
Tout d'abord, nous allons lancer un premier Pod, qui contiendra simplement un site web affichant une page.

Un Pod, c'est plus ou moins la version Kubernetes d'un conteneur de Docker.

- Pour créer ce Pod, il faut créer un Deployment et préciser l'image (Docker en l'occurence) utilisée pour créer le Deployment.
- La commande suivante vous permet de créer un Deployment :
```
kubectl create deployment [Nom du Deployment] --image=[chemin/vers/l'image/sur/Docker/Hub:tag]
```
- Pour récupérer des images, il est possible de les publier sur [Docker Hub](https://hub.docker.com). Celle que nous allons utiliser se trouve à l'adresse https://hub.docker.com/xhelozs/csc8567. Elle porte le tag "v1".
- Les informations sur la construction de l'image sont disponibles dans le dossier `csc8567-web-nodb` de ce même répo.
- Comparez votre objectif à la documentation pour réussir à créer le Pod : [votre premier Deployment](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/).
- Ensuite, nous allons utiliser le `port-forward` permis par Kubernetes pour mapper le port du Pod du site sur un port de notre interface `localhost`.
```
kubectl port-foward pods/[Nom du Pod] [Port localhost]:[Port du Pod]
```
Alors, le site devrait être visible depuis `localhost:[Port localhost]`. Si c'est le cas, vous avez complété ce Contenu avec succès !

### Questions

Le schéma déjà ça sera bien ! Venez voir un prof quand vous (pensez) l'avoir fini.

## Deuxièmes pas sur Kubernetes (ça ne se dit pas ?) (Défi 2)

### Contenu

Bon, on a déployé un Pod, c'est sympa, mais ça nous avance pas beaucoup de Docker, c'est même plus compliqué...

Vous en faites pas : on va compliquer encore un peu plus les choses !

Vous allez créer maintenant votre premier Deployment, pour l'image `csc8567` utilisée lors du dernier Défi.

**Votre Deployment créera 3 répliques du Pod faisant tourner le site.**

Pour ce faire, il va falloir comprendre ce qu'est un Deployment, comment ça se configure. Pour ceci, vous pouvez visionner [cette vidéo](https://youtu.be/qmDzcu5uY1I?si=jeoMTcyKxxQ70jmG).

Ensuite, le soucis précédent va se réitérer : il faut trouver un moyen d'accéder au service. On va utiliser pour ça une combinaison gagnante : un service NodePort + un proxy.

Deux ressources sont alors intéréssantes à parcourir pour mieux comprendre :
- [Accéder à des services qui tournent sur le Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/)
- [Construire des URLs personnalisés sur l'API](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/#manually-constructing-apiserver-proxy-urls)

Ensuite, on souhaite allouer des ressources particulières à chaque Pod du Deployment. En particulier :
- 1/10 CPU par pod 
- 100 Mo de mémoire RAM par pod
En revanche, on veut aussi limiter les ressources à :
- 1/5 CPU par pod
- 200 Mo de mémoire RAM par pod

Cette ressource va vous aider à arriver à vous fins :
- [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

Vous pourrez alors vous connecter à votre service via le proxy :
```bash
kubectl proxy
```
Votre service devrait alors être disponible à l'adresse : 
`http://127.0.0.1:8001/api/v1/namespaces/[votre espace de noms]/services/[nom de votre service]/proxy/`

### Questions
1. Quel est le but d'un service ?
2. Quelle est la différence entre les service ClusterIP et NodePort ?
3. Et le schéma !!

## Connexions dangereuses (Défi 3)
### Contenu
Pas mal ! Maintenant l'étape suivante : on va reprendre votre projet Django.

Avant tout, vous aller le copier, et faire en sorte que **le site soit contenu avec les deux apps (public et api) dans une image Docker**, que vous allez publier sur [Docker Hub](https://hub.docker.com).

Utilisez un Deployment pour déployer votre site Django similairement à celui créé précédemment.

Sauf que c'est pas fini cette fois ! Vous allez aussi créer un autre Deployment, pour déployer la base de données Postgresql. Le service que vous utiliserez est un ClusterIP.

Et un peu plus de documentation pour comprendre comment faire !

- [Connecter des Applications avec des Services](https://kubernetes.io/docs/tutorials/services/connect-applications-service/)

### Questions

- Pourquoi utilise-t-on un service type NodePort pour le site Django et un service type ClusterIP pour la base de données ?
- Quelle critique pouvez-vous donner vis-à-vis de l'utilisation de Pods pour la base de données ?
- Le schéma !

## Internet ! Me voilà ! (Défi 4)
### Contenu
Vraiment pas mal !

Maintenant, on va faire en sorte que votre site soit accessible depuis Internet. Plus partique pour un site web, non ?

Vous allez donc créer ce que l'on appelle un Ingress.

Le but, c'est que votre site devienne accessible à l'adresse http://django.votre_espace_de_noms.kube.luxbulb.org/

Ici, des [généralités sur les Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/).

### Questions

On met à jour le schéma, ça suffira !

## Au complet ! (Défi 5)
### Contenu

C'est enfin le moment : on va reproduire l'infrastructure qu'on avait sur Docker, mais version (presque) Kubernetes !

Pour récapituler, il faut que :
- Vous ayez deux images Docker permettant de faire tourner séparément les applications API et Public (ou l'équivalent de votre projet)
- Vous faites tourner ces deux images dans des Deployments (3 pods répliqués, même allocations/limitations de ressources qu'au Défi 2), derrière des services bien choisis
- Vous faites tourner la base de données via un Deployement (3 pods répliqués, allocations/limitations idem que pour API et Public), derrière un service bien choisi
- Vous utilisez l'Ingress pour diriger les requêtes (l'Ingress vient remplacer votre proxy Nginx de Docker)

Pas plus de documentation pour cette fois ! Vous avez déjà tout ce qu'il vous faut.

### Questions

Un beau schéma !

## Quelqu'un a dit "HELM" ?! (Défi 6)

### Contenu

Vous vous souvenez de docker compose ? Eh bien on a un peu l'équivalent sur Kubernetes : Helm, et surtout les chartes Helm.

L'objectif est de créer une charte Helm qui automatise le déploiement de l'infrastructure que vous avez mise en oeuvre au Défi précédent.

Également, vous aller utiliser ConfigMaps pour stocker les informations de nom d'hôte et de port de la base de données.

Cette fois, on a de la doc à vous partager pour tout ça :

- [Bien débuter avec Helm](https://helm.sh/docs/chart_template_guide/getting_started/)
- [Utiliser ConfigMaps dans Kubernetes](https://kubernetes.io/docs/concepts/configuration/configmap/)

### Questions

Rien ! Une belle charte Helm fera amplement l'affaire.

**À PARTIR DE CE DÉFI, SAUF MENTION CONTRAIRE, VOUS DEVEZ METTRE À JOUR VOTRE CHARTE HELM POUR CHAQUE NOUVEL AJOUT OU NOUVELLE MODIFICATION À VOTRE INFRASTRUCTURE !!**



## Connexions moins dangereuses (Défi 7)

Si vous avez réussi le Défi précédent, allez voir un professeur pour qu'il valide votre réussite et révèle ce défi.

**???**

## **???** (Défi 8)

**???**
