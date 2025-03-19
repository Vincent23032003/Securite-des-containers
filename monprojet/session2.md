# Session 2: Bonnes Pratiques de Sécurité

## Objectif
Appliquer les meilleures pratiques pour sécuriser les containers.
Comprendre les principes de gestion des droits et permissions.

---

## Activités Pratiques

### 1. Éviter l’Exposition Involontaire de Ports

1. **Lancer un container avec restriction de ports** 

```bash
docker run -d -p 8080:80 nginx
```
Docker a bien démarré un conteneur Nginx en mode détaché (-d), et a associé le port 8080 de ton hôte au port 80 du conteneur. 

![portRestrictions](images/session2/portRestrictions.png) 

La longue suite de caractères que l'on peut apercevoir correspond à l’ID du conteneur qui vient d’être créé.