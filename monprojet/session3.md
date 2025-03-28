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


On a crée le pod. 

![image](https://github.com/user-attachments/assets/2d63229a-10f7-404e-b423-ca7861679442)

On afficher les logs grâce à cette commande : 
```bash
kubectl logs nginx -n test-rbac
```

Les logs nous permettent d'avoir des informations sur les scripts de configurations ou encore sur les informations de démarrage de NGINX.
