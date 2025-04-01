# Session 4 : Sécurité dans la CI/CD

## Objectif
Comprendre l’importance de l’intégrité des images

## Activités Pratiques

Avant toute chose, nous devons installer et vérifier quelques outils
Tout d'avord, nous devons installer Cosign. Pour cela, nous utilisons les commandes suivantes :


```bash
curl -sSL https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 -o cosign
chmod +x cosign
sudo mv cosign /usr/local/bin/
```
<img width="428" alt="image" src="https://github.com/user-attachments/assets/edf185db-f860-4099-a487-4d25a1b94483" />

On vérifie que l'installation a bien fonctionné :

<img width="345" alt="image" src="https://github.com/user-attachments/assets/2ff0d356-1756-4842-bdee-f5331b58e14a" />

On voit que Cosign a été installer avec succès.

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

On utilise la commande suivante pour générer une clé RSA :

```bash
gpg --full-generate-key  # Choisir "RSA (sign only)" et une passphrase forte
```
On voit que cela bien créé une clé RSA :

<img width="469" alt="image" src="https://github.com/user-attachments/assets/7ade4d2a-32fa-4fd9-baa8-d56cde911107" />

On entre une passphrase forte :

![image](https://github.com/user-attachments/assets/fe09a2eb-b4ad-4981-9224-e7b5c9db242d)

On utilise la commande suivante pour noter la clé de l'id :
```bash
gpg --list-secret-keys   # Noter l'ID de la clé (ex: `ABCD1234`)
```
<img width="289" alt="image" src="https://github.com/user-attachments/assets/bf94d645-c39e-46d8-a6f9-b12016935e24" />

On voit que l'id de la clé est '947CDDEC1976107C27F1EAF494C065C4D6F17667'.

### 2. Exporter la clé privée GPG pour Cosign

```bash
gpg --export-secret-keys --armor ABCD1234 > private-gpg.key
```
![image](https://github.com/user-attachments/assets/217267d6-af35-4963-a44f-8bdfd7a42a3c)

Cela nous demande notre passphrase.

![image](https://github.com/user-attachments/assets/903400ee-50b9-4153-8e3d-e030e36f0577)


### 3. Signer une image Docker et la pousser vers GitLab

Avant toute chose, on créer un projet dans gitlab.

![image](https://github.com/user-attachments/assets/9073edb7-0ab5-47b8-a0c3-4f1265f43b85)
Projet créé avec succès.

1. **Build et tag de l'image**

On utilise la commande suivante pour build et tag l'image :
```bash
docker build -t registry.gitlab.com/groupe-jvq/tp4-securite-containers/image:v1 .
```

![image](https://github.com/user-attachments/assets/a94eca6a-e2fd-455c-99c1-30505efd8e51)



2. **Login dans le registry de gitlab avec un personal access token**

On a besoin de créer un token. 
![image](https://github.com/user-attachments/assets/9ca083ec-8377-4cab-bb6c-1bfeb7ac2193)
On affecte un nom et une date d'expiration (ici le 01/08/2025).

```bash
docker login registry.gitlab.com
```

![image](https://github.com/user-attachments/assets/26d07f19-7860-48cb-97fe-2415e9cec106)

3. **Push de l'image**

On récupère l'id de l'image :
![image](https://github.com/user-attachments/assets/c9a23b0a-c30f-49fb-b58e-da91f624be04)


On utilise cette commande pour push cette image :

```bash
docker push registry.gitlab.com/votre-projet/image:v1
```

4. **Installation de cosign**
```bash
curl -sSfL https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 -o cosign && chmod +x cosign
```


5. **Signature avec Cosign + clé GPG**

```bash
./cosign sign --key gpg://private-gpg.key registry.gitlab.com/votre-projet/image:v1
```

### 4. Modifier l'image et pousser une version modifiée
