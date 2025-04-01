# Session 4 : Sécurité dans la CI/CD

## Objectif
Comprendre l’importance de l’intégrité des images

## Activités Pratiques

### 1. Générer une paire de clés GPG

On utilise la commande suivante :

```bash
gpg --full-generate-key  # Choisir "RSA (sign only)" et une passphrase forte
gpg --list-secret-keys   # Noter l'ID de la clé (ex: `ABCD1234`)
```

### 2. Exporter la clé privée GPG pour Cosign

```bash
gpg --export-secret-keys --armor ABCD1234 > private-gpg.key
```

### 3. Signer une image Docker et la pousser vers GitLab

1. **Build et tag de l'image**
```bash
docker build -t registry.gitlab.com/votre-projet/image:v1 .
```

2. **Login dans le registry de gitlab avec un personal access token**
```bash
docker login registry.gitlab.com
```
3. **Push de l'image**
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
