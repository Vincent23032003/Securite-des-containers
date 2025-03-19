# Session 1: Container Security - Activités Pratiques

## Objectif
Cette session a pour but de vous familiariser avec les bases des containers Docker, leur fonctionnement, et les ressources qu'ils consomment.

---

## Activités Pratiques

### 1. Lancer un Container Simple
1. **Installer Docker**  
    Si Docker n'est pas encore installé, suivez le [guide officiel d'installation](https://docs.docker.com/get-docker/).

2. **Exécuter un container de test**  
    ```bash
    docker run --rm hello-world
    ```
    - Observez les logs affichés.
    - Comprenez les étapes du lancement du container.  
      Lorsque vous exécutez la commande `docker run --rm hello-world`, Docker télécharge l'image `hello-world` si elle n'est pas déjà présente localement, puis crée et exécute un container basé sur cette image. Une fois le container terminé, il est automatiquement supprimé grâce à l'option `--rm`.

    ![hello-world](images/session1/hello-word.png)  
    *Illustration de l'exécution réussie du container `hello-world` et des logs affichés.*

---

### 2. Explorer un Container en Interactif
1. **Lancer un container interactif basé sur Alpine**  
    ```bash
    docker run -it --rm alpine sh
    ```
2. **Tester des commandes Linux dans le container**  
    - `ls`  
        - Cette commande liste les fichiers et dossiers présents dans le répertoire actuel.

    - `pwd`  
        - Cette commande affiche le chemin absolu du répertoire courant.

    - `whoami`  
        - Cette commande indique l'utilisateur actuellement connecté dans le container.

        ![alpine-start](images/session1/alpine-start.png)  
        *Illustration de l'exécution des commandes de base dans un container Alpine.*

        ![Ping google](images/session1/ping-google.png)  
        *Exemple de test de connectivité réseau en utilisant la commande `ping` pour vérifier l'accès à Google.*

---

### 3. Analyser les Ressources Système d’un Container
1. **Lancer un container et surveiller ses ressources**  
    ```bash
    docker run -d --name test-container nginx
    docker stats test-container
    ```
2. **Observer la consommation de CPU et mémoire**.

    ![CPU-stats](images/session1/CPU-stats.png)  
    *Illustration de la consommation des ressources système (CPU et mémoire) d’un container en cours d’exécution.*

---

### 4. Lister les Capacités d’un Container
1. **Vérifier les permissions**  
    ```bash
    docker run --rm --cap-add=SYS_ADMIN alpine sh -c 'cat /proc/self/status'
    ```
    ![permission1](permission1.png)
    ```markdown
    *Illustration des permissions et capacités d’un container exécuté avec l’option `--cap-add=SYS_ADMIN`.*  
    *La première image montre les permissions par défaut, tandis que la seconde illustre les permissions étendues obtenues grâce à cette option.*  
    Le fichier `/proc/self/status` contient des informations détaillées sur le processus en cours d'exécution.  
    L’option `--cap-add=SYS_ADMIN` accorde au container des privilèges supplémentaires, permettant d'effectuer des actions normalement restreintes.
    ```

---

## Notes
- Assurez-vous d'avoir les droits administratifs pour exécuter certaines commandes.
- Consultez la documentation Docker pour plus de détails : [Docker Documentation](https://docs.docker.com/).
