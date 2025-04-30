# Session 4 : Sécurité dans la CI/CD

## Objectif
Comprendre l’importance de l’intégrité des images

## Activités Pratiques

Avant toute chose, nous devons installer et vérifier quelques outils
Tout d'avord, nous devons installer Cosign. Pour cela, nous utilisons les commandes suivantes :


```bash
curl -L -o cosign.exe https://github.com/sigstore/cosign/releases/download/v2.2.3/cosign-windows-amd64.exe
```

Pus  on déplace le fichier téléchargé dans un dossier PATH

```bash
move cosign.exe C:\Windows\System32\
```

Enfin, on vérife  l'installation
```bash
cosign version
```

![image](https://github.com/user-attachments/assets/e5404a86-0df3-4063-aa89-fc682b34021c)


Mais qu'est-ce que Cosign ?

Cosign est un utilitaire open source en ligne de commande permettant de signer et vérifier des images de conteneurs (Docker/OCI).
Cosign utilise une clé (comme GPG) pour signer des images Docker et stocke cette signature dans le registre d’images.
Il peut également s'intégrer dans des pipelines CI/CD.

---

On vérifie ensuite que GPG est bien installé. Normalement Kali a GPG préinstallé.

```bash
gpg --version
```

<img width="293" alt="image" src="https://github.com/user-attachments/assets/d2550296-996e-4a97-b849-09801cdada32" />

On voit que GPG est bien préinstaller.

Mais qu'est-ce que GPG ?

GNU Privacy Guard est un système qui permet de créer et utiliser des clés cryptographiques (asymétriques).
On va s'en servir pour signer numériquement des images ou du code et garantir l’identité de l’auteur + l’intégrité du contenu.

---

### 1. Générer une paire de clés GPG
#### 1.1 Génération paire de clés 

On utilise la commande suivante pour générer une clé RSA :

```bash
./cosign generate-key-pair
```
On voit que cela bien créé une clé RSA :

![image](https://github.com/user-attachments/assets/04954fe7-9bf5-42cf-bcdf-a2ba463ad4a6)

On entre bien entendu un mot de passe fort.

On voit que les fichiers cosign.key et cosign.pub ont été créés.

cosign.key → c’est ta clé privée 
cosign.pub → c’est ta clé publique

![image](https://github.com/user-attachments/assets/6de1c1dc-4d75-4b05-a111-6504e5906b27)



On utilise la commande suivante pour noter la clé de l'id :
```bash
gpg --list-secret-keys   # Noter l'ID de la clé (ex: `ABCD1234`)
```

Par mesure de sécurite, je ne dévoilerai pas l'id de ma clé.

<img width="305" alt="image" src="https://github.com/user-attachments/assets/0dfaeb60-4a89-426e-ab30-5b440ad930a5" />



Cependant, on a préalablement installé GPG avec l'extension Kleopatra.

![image](https://hackmd.io/_uploads/ByfnssXJee.png)


#### 1.2 Exportation de la paire de clés 

```bash
gpg --export-secret-keys --armor  > ma_cle_publique.asc
```

le fichier a bien été créé : 
![image](https://hackmd.io/_uploads/S1oLnjXklg.png)



![image](https://github.com/user-attachments/assets/903400ee-50b9-4153-8e3d-e030e36f0577)


### 2. Signer une image Docker et la pousser vers GitLab

Avant toute chose, on créer un projet dans gitlab.

![image](https://hackmd.io/_uploads/HJ_Hy2Q1gl.png)

Projet créé avec succès.

1. **Build et tag de l'image**

On utilise la commande suivante pour build et tag l'image :
```bash
docker build -t registry.gitlab.com/groupe-jvq/tp-4/image:v1 .
```

![image](https://github.com/user-attachments/assets/f02468ca-f448-4ca9-bfc9-989e11c992ea)



2. **Login dans le registry de gitlab avec un personal access token**

On a besoin de créer un token. 
![image](https://github.com/user-attachments/assets/9ca083ec-8377-4cab-bb6c-1bfeb7ac2193)
On affecte un nom et une date d'expiration (ici le 01/08/2025).

```bash
docker login registry.gitlab.com
```

![image](https://github.com/user-attachments/assets/2e3c1f3e-df1b-48c1-97e7-f9670d895e13)


3. **Push de l'image**

On récupère l'id de l'image :
![image](https://github.com/user-attachments/assets/c9a23b0a-c30f-49fb-b58e-da91f624be04)


On utilise cette commande pour push cette image :

```bash
docker push registry.gitlab.com/vincent.bare.2003/tp-4/image:v1
```
![image](https://github.com/user-attachments/assets/0ed68564-4095-4852-b6b2-eb3823830546)



Nous vérifions que Container registry est bien activé dans Visibility, project features, permissions.

![image](https://hackmd.io/_uploads/BkIomhmyxl.png)

J'ai trouve mon erreur, c'était au moment où j'ai créé le projet, j'avais laissé l'url par défaut https://gitlab.com/group-jvq, qui était en l'occurrence "groupe-jvq". J'ai donc récréer un projet en mettant comme porject URL https://gitlab.com/vincent.bare.2003

![image](https://hackmd.io/_uploads/SJPAKn7Jel.png)

J'ai donc refait la manip et cela a bie marché :


![image](https://hackmd.io/_uploads/B1C8q3X1el.png)

![image](https://hackmd.io/_uploads/ry65Yh7Jel.png)

On signe l'image avec Cosign : 

![image](https://hackmd.io/_uploads/ryQzChmkeg.png)

On vérifie que cela a bien marché : 
![image](https://hackmd.io/_uploads/Sk4SRn7kxe.png)


Cela a bien fonctionné !


On signe maintenant avec Cosign et la clé gpg :

```bash
./cosign sign --key cosign.key registry.gitlab.com/vincent.bare.2003/tp-4/image:v1
```
![image](https://github.com/user-attachments/assets/556913fc-93c0-4eed-b4e6-72f5220417d9)




### 3. Modifier l'image et pousser une version modifiée

1. **Créer une modification**

On exécute cette commande pour lancer un conteneur dans l'image v1 :

```bash
docker run -it --rm registry.gitlab.com/vincent.bare.2003/tp-4/image:v1 touch /tampered
```

![image](https://github.com/user-attachments/assets/2ff3ab4c-6530-4e93-9e87-a87db36129d4)

2. Committer la modification et pousser la nouvelle version

2.1 **Créer une nouvelle image Docker avec la modification**

Ensuite, il faut committer cette modification pour créer une nouvelle image Docker et la pousser avec un nouveau tag (v2).

```bash
docker commit $(docker ps -lq) registry.gitlab.com/vincent.bare.2003/tp-4/image:v2
```
![image](https://github.com/user-attachments/assets/5eb51e85-1910-4973-bab7-1d33c091b746)


2.2 **Pousser la nouvelle version vers GitLab**

```bash
docker commit $(docker ps -lq) registry.gitlab.com/vincent.bare.2003/tp-4/image:v2
```

![image](https://github.com/user-attachments/assets/feb2f529-1cc0-4e75-8bbb-029866790b93)



4. Vérifier la signature avant/après modification

1. **Vérification de l'image originale**

```bash
./cosign verify --key cosign.pub registry.gitlab.com/vincent.bare.2003/tp-4/image:v1
```
![image](https://github.com/user-attachments/assets/e75388e0-07c9-4782-9aae-4d8bb290ba6f)

Cela marche bien, ce qui signifie que l'image v1 a été signée correctement avec Cosign auparavant.

2. **Vérification de l'image altérée **

```bash
./cosign verify --key cosign.pub registry.gitlab.com/vincent.bare.2003/tp-4/image:v2
```

<img width="373" alt="image" src="https://github.com/user-attachments/assets/cd07befc-b9fd-4b50-8a58-d55d863e43d7" />

Cette vérification doit échouer, car l'image a été modifiée (/tampered a été ajouté) après qu'elle ait été signée initialement, ce qui signifie que la signature de l'image v1 ne correspondra pas à celle de l'image v2.

5. Quelles sont les résultats de ces 2 commandes ?

```bash
./cosign verify --key cosign.pub registry.gitlab.com/vincent.bare.2003/tp-4/image:v1
```
Cela a bien marché comme on a pu voir plus haut dans la question 4.
Cela signifie que l'image v1 est toujours authentique et n'a pas été altérée depuis qu'elle a été signée.


```bash
./cosign verify --key cosign.pub registry.gitlab.com/vincent.bare.2003/tp-4/image:v2
```
Résultat attendu :
Cette commande devrait échouer, car l'image a été modifiée après la signature initiale.



### 1. Créer un pipeline CI/CD avec Gitlab CI

#### 1.1 Créer le fichier Dockerfile

![image](https://github.com/user-attachments/assets/a5dfbe4e-56a5-4a0e-ad35-61115c68b1bf)


#### 1.2 Créer le fichier .gitlab-ci.yml

![image](https://github.com/user-attachments/assets/fcff7bca-faba-4068-b8c0-554f1263837c)

#### 1.3 Faire un push du fichier .gitlab-ci.yml

```bash
git add .gitlab-ci.yml
git commit -m "Ajout du pipeline CI/CD"
git push
```

![image](https://github.com/user-attachments/assets/58ea556b-5dcc-448a-94f9-df145a2c27f0)
