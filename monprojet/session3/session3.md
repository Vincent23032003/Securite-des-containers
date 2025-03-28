# Session 3: Bonnes Pratiques de Sécurité

## Activités Pratiques

### 1. Déployer un Cluster Kubernetes avec Kind

![image](https://github.com/user-attachments/assets/dba8f45b-c61e-43c9-8e09-84a8f214c4d6)

Kind et Kubernetes sont bien installés avec la version 0.27.0 et 1.30.5 respectivement.

![image](https://github.com/user-attachments/assets/70417b7c-aeba-44fe-b007-182bb7a75899)

![image](https://github.com/user-attachments/assets/220e17fe-44b1-4b29-a807-08836223aecf)

On crée le cluster kind et on vérifie si cela a bien fonctionné.

On observe que les 2 master nodes et les 2 worker nodes sont bien crées. 

![image](https://github.com/user-attachments/assets/4c7442a2-eb28-4c4d-86ec-1f86d84ebe3c)

On a 5 namespaces de créer qui sont installés par défaut.

Ces namespaces vont nous permettre d'isoler des ressources au sein de notre cluster.

Il est également possible de gérer les accès pour un certain namespace grâce aux politiques de contrôle d'accès basées sur les rôles (RBAC), ce que nous allons faire juste après.

### 2. Expérimentation des RBAC

![image](https://github.com/user-attachments/assets/68883ed5-f253-4e72-894a-f31bcaf6f66b)

Le namespace 'test-rbac' est bien crée.

On va donc travailler dans ce namespace pour appliquer les RBACs.

![image](https://github.com/user-attachments/assets/5d2a03ef-a712-4348-b214-8f7a1ee2a765)

![image](https://github.com/user-attachments/assets/ab2e9793-df4e-4a39-a890-dff08365e513)


On a crée le pod test-rbac. 

![image](https://github.com/user-attachments/assets/2d63229a-10f7-404e-b423-ca7861679442)

On afficher les logs grâce à cette commande : 
```bash
kubectl logs nginx -n test-rbac
```

Les logs nous permettent d'avoir des informations sur les scripts de configurations ou encore sur les informations de démarrage de NGINX. 

Le pod est donc déployé avec succès. 

![image](https://github.com/user-attachments/assets/f5b57785-0c66-419b-84d0-310c2b2c6adb)

![image](https://github.com/user-attachments/assets/f37bf00c-7b90-460f-81f9-580d4444ee8e)

On déploie le rôle qu'on va appliquer à un utilisateur. On définit les rôles "read" et "list". L'utilisateur est donc autorisé à lire et à lister la ressource. 

Pour afficher ce rôle on utilisa la commande :
```bash
kubectl get roles -n test-rbac
```

Dans une situation réelle il est important de définir le minimum de rôles qu'a besoin un utilisateur pour éviter qu'il puisse accéder à des ressources sensibles.

![image](https://github.com/user-attachments/assets/40a8a8d3-b6ef-48fc-9b50-3c73a28e6f21)

On applique le fichier role binding pour lier le role crée précedemment à l'utilisateur qeu l'on va créer.

![image](https://github.com/user-attachments/assets/3facc9dd-a587-46a8-8121-11119ffd0f8c)

On copie le certificat d'autorité et la clé privée de l'autorité de certification du conteneur "kind-control-panel" qui est le node master dans notre cluster.

On a besoin de ces fichiers car ils vont nous servir pour générer le certificat d'autorité pour le nouvel utilisateur. En effet, la clé privée va nous permettre de signer le nouveau certificat. 

![image](https://github.com/user-attachments/assets/673232b4-7850-448d-9158-04dc001de8fd)

![image](https://github.com/user-attachments/assets/a4adee5f-9c7d-4323-aefb-022f803d0508)

On crée la clé qui va nous servir pour générer la demande de certificat (titi.csr). 

Par la suite on génère le certificat d'autorité pour titi qui va l'authentifier et vérifier son identité auprès du cluster. 

Les certificats ont plusieurs avantages en termes de sécurité : 
- Les certificats permettent d'établir des communications chiffrées entre les différents composants du cluster
- En utilisant des certificats, vous pouvez appliquer des politiques RBAC strictes. Cela permet de définir des différentes permissions, limitant ainsi les actions que chaque utilisateur peut effectuer.
- Les certificats permettent de tracer les actions effectuées par chaque utilisateur, facilitant ainsi l'audit et la détection des activités suspectes.
- En limitant l'accès au cluster aux seules entités possédant les certificats appropriés, vous réduisez le risque d'accès non autorisé et de compromission du cluster.

Il est donc essentiel d'utiliser des certificats pour l'authentification des utilisateurs.

Cependant, en temps normal les certificats doivent être signés par des autorités de certification comme DigiCert pour assurer l'identité des identités présentes sur le web. 

Dans notre cas, nous avons un certificat auto-signé ce qui n'a aucune valeur sur le web.

![image](https://github.com/user-attachments/assets/6907b093-f46c-4d75-a241-549102a944cc)

Contenu du certificat de l'utilisateur titi. 

![image](https://github.com/user-attachments/assets/d527bb24-e20b-407f-b5a3-24d8fd73cb60)

Avant d'ajouter titi dans le kube context, on doit copier les fichiers de certificat et de clé dans le répertoire .kube.

![image](https://github.com/user-attachments/assets/b08c32e9-e62a-4789-8652-5c92f7d017aa)

Titi est donc bien ajouté dans le kube context avec les bons fichiers de clé et de certificat.

![image](https://github.com/user-attachments/assets/b2c19fe9-fb98-47f1-9b59-3b234b883b00)

On crée le contexte pour le nouvel utilisateur dans notre namespace afin de pouvoir se connecter.

![image](https://github.com/user-attachments/assets/7decf4e8-d33f-441c-9aef-a3566ea772eb)

On change de contexte pour être connecté en tant que titi et on liste les pods qui sont en train d'être exécutés, ce qui est autorisé par le rôle configuré précédemment.

![image](https://github.com/user-attachments/assets/696f4b5e-bd1f-4edf-97e0-32b167dd9c2b)

Il est interdit de créer un pod pour titi car nous avions spécifié uniquement les actions "get" et "list" dans le fichier role-pod-reader qui définit les actions que peut faire titi. 

Si nous voulons que titi puisse créer un pod il faut rajouter l'action "create".

![image](https://github.com/user-attachments/assets/74f00f70-f9aa-4788-bb52-9cdd83731687)

On change de contexte.

### 3. Détection et alerte d'intrusions dans kubernetes avec l'outil Falco

Falco est un outil de sécurité open-source conçu pour détecter les comportements anormaux et les menaces en temps réel dans les environnements Kubernetes.

Il permet notamment de : 
- Détecter des menaces en temps réel
- Alerter et de notifier en cas d'intrusion
- Définir des politiques de sécurité

![image](https://github.com/user-attachments/assets/9355b35f-b5e3-4ec9-995a-1f478f812dcc)

On ajoute le repo heml et on update.

Helm est un gestionnaire de paquets pour Kubernetes on peut le comparer à apt pour Linux. Il simplifie la définition, l'installation et la gestion des applications Kubernetes complexes.

![image](https://github.com/user-attachments/assets/f90b2cc2-bfdd-4dce-8a48-ca51f40f27df)

![image](https://github.com/user-attachments/assets/1a6212a5-c288-420d-b222-f785c14265f4)

On installe falco et on vérifie que les pods sont bien créés.

![image](https://github.com/user-attachments/assets/cad427ba-8710-4558-b332-8e6a011c17d6)

![image](https://github.com/user-attachments/assets/7c79d832-5d0a-4ede-b236-fd6909a47455)

On démarre l'ui de falco et on y accède sur notre navigateur. 

### 4. Falco en pratique

![image](https://github.com/user-attachments/assets/9cc00645-28ef-404d-888d-07f308b68efc)

On crée de l'activité dans le cluster avec le pod 2.

![Enregistrement de lécran 2025-03-28 190812 (1)](https://github.com/user-attachments/assets/53ab725b-2a96-4662-b09c-82b1ae672535)

Comme on peut le voir dans la vidéo, on accède un shell dans le pod en étant root directement, ce qui nous donne tous les droits possibles. Les attaquants peuvent donc exploiter cette faille de sécurité pour compromettre le pod.

Cela nous crée une alerte dans falco également qui nous alerte sur la situation.

![Enregistrement de lécran 2025-03-28 193337](https://github.com/user-attachments/assets/7afea17a-4ad7-47e9-bd2d-e158cafbc541)

On génère une requête sur l'API Kubernetes pour créer du traffic ce qui est reporté dans falco. Sa priorité est notice, falco considère que c'est une action bénigne et la règle est la suivante : Contacter le serveur API K8S à partir d'un conteneur.
