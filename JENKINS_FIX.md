# Solution au problème Node.js

## Problème
L'erreur `Node.js n'est pas installé` se produit car l'agent Jenkins n'a pas Node.js installé localement.

## Solution appliquée
Le Jenkinsfile a été modifié pour utiliser des **agents Docker** avec Node.js préinstallé pour les stages qui nécessitent npm :

- **Install Dependencies** : Utilise `node:20-alpine`
- **Lint** : Utilise `node:20-alpine`
- **Build Application** : Utilise `node:20-alpine`
- **Build Docker Image** : Utilise l'agent principal (nécessite Docker)
- **Test & Deploy** : Utilisent l'agent principal (nécessitent Docker)

## Prérequis Jenkins

### 1. Plugin Docker Pipeline
Installez le plugin **Docker Pipeline** dans Jenkins :
\`\`\`
Manage Jenkins → Plugins → Available → Rechercher "Docker Pipeline" → Install
\`\`\`

### 2. Docker sur l'agent Jenkins
Assurez-vous que Docker est installé et accessible sur votre agent Jenkins :
\`\`\`bash
docker --version
docker ps
\`\`\`

### 3. Permissions Docker
L'utilisateur Jenkins doit avoir les permissions Docker :
\`\`\`bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
\`\`\`

## Avantages de cette approche

✅ **Pas besoin d'installer Node.js** sur l'agent Jenkins  
✅ **Version Node.js garantie** (toujours 20-alpine)  
✅ **Environnement isolé** pour chaque build  
✅ **Cache réutilisable** avec `reuseNode true`  
✅ **Plus rapide** que l'installation manuelle de Node.js

## Test du pipeline

1. Commitez le nouveau Jenkinsfile
2. Lancez un build dans Jenkins
3. Vérifiez que tous les stages passent au vert

## Dépannage

### Erreur "Cannot connect to Docker daemon"
\`\`\`bash
# Vérifier que Docker fonctionne
sudo systemctl status docker

# Redémarrer Docker si nécessaire
sudo systemctl restart docker
\`\`\`

### Erreur "Permission denied" sur Docker
\`\`\`bash
# Ajouter l'utilisateur Jenkins au groupe docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
\`\`\`

### Image Docker trop lente à télécharger
Le premier build sera plus lent car Jenkins doit télécharger l'image `node:20-alpine`. Les builds suivants seront beaucoup plus rapides grâce au cache.
