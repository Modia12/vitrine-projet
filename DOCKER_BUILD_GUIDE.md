# Guide de résolution des problèmes Docker Build

## Problèmes courants et solutions

### 1. Permission denied lors du build Docker

**Symptôme:** `permission denied while trying to connect to the Docker daemon socket`

**Solutions:**
\`\`\`bash
# Ajouter l'utilisateur Jenkins au groupe docker
sudo usermod -aG docker jenkins

# Redémarrer Jenkins
sudo systemctl restart jenkins

# Vérifier les permissions
sudo chmod 666 /var/run/docker.sock
\`\`\`

### 2. Docker n'est pas disponible

**Symptôme:** `command not found: docker`

**Solutions:**
\`\`\`bash
# Installer Docker sur l'agent Jenkins
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Démarrer Docker
sudo systemctl start docker
sudo systemctl enable docker
\`\`\`

### 3. Erreur "no space left on device"

**Symptôme:** Build échoue avec erreur d'espace disque

**Solutions:**
\`\`\`bash
# Nettoyer les images Docker non utilisées
docker system prune -a -f

# Nettoyer les volumes
docker volume prune -f

# Vérifier l'espace disque
df -h
\`\`\`

### 4. Build multi-stage échoue

**Symptôme:** Erreur pendant le build des stages

**Solutions:**
- Vérifier que `next.config.mjs` contient `output: 'standalone'`
- S'assurer que toutes les dépendances sont dans `package.json`
- Vérifier les logs avec `--progress=plain`

### 5. L'image se construit mais l'application ne démarre pas

**Symptôme:** Container démarre puis s'arrête immédiatement

**Solutions:**
\`\`\`bash
# Vérifier les logs du conteneur
docker logs test-spray-info

# Tester manuellement
docker run -it --rm spray-info:latest sh
node server.js

# Vérifier que le fichier server.js existe
docker run --rm spray-info:latest ls -la
\`\`\`

### 6. Erreur "Cannot find module 'next'"

**Symptôme:** Module Next.js non trouvé dans le conteneur

**Solutions:**
- Vérifier que les `node_modules` sont correctement copiés
- S'assurer que `npm ci` s'exécute correctement dans le Dockerfile
- Utiliser `npm install` au lieu de `npm ci` si nécessaire

## Configuration Jenkins recommandée

### Plugins nécessaires
- Docker Pipeline
- Docker Commons
- Pipeline

### Variables d'environnement Jenkins
\`\`\`groovy
environment {
    DOCKER_IMAGE = 'spray-info'
    DOCKER_TAG = "${env.BUILD_NUMBER}"
    DOCKER_BUILDKIT = '1'  // Active BuildKit pour de meilleures performances
}
\`\`\`

### Configuration Docker dans Jenkins

1. **Aller dans:** Manage Jenkins > Configure System
2. **Ajouter Docker Cloud** (si nécessaire)
3. **Configurer les credentials Docker Hub** (si push vers registry)

## Test manuel du Dockerfile

\`\`\`bash
# Construire l'image
docker build -t spray-info:test .

# Tester l'image
docker run -d -p 3000:3000 --name test-app spray-info:test

# Vérifier les logs
docker logs -f test-app

# Tester l'application
curl http://localhost:3000

# Nettoyer
docker stop test-app
docker rm test-app
\`\`\`

## Optimisations possibles

### 1. Utiliser le cache Docker
\`\`\`groovy
sh """
    docker build \
        --cache-from ${DOCKER_IMAGE}:latest \
        -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
        .
"""
\`\`\`

### 2. Build parallèle avec BuildKit
\`\`\`groovy
environment {
    DOCKER_BUILDKIT = '1'
}
\`\`\`

### 3. Réduire la taille de l'image
- Utiliser `.dockerignore` pour exclure les fichiers inutiles
- Nettoyer le cache npm dans le Dockerfile
- Utiliser des images Alpine

## Vérification de la configuration

\`\`\`bash
# Vérifier que Docker fonctionne
docker --version
docker ps

# Vérifier les permissions
groups jenkins

# Vérifier l'espace disque
df -h /var/lib/docker

# Tester le build localement
docker build -t test .
\`\`\`

## Support

Si les problèmes persistent:
1. Vérifier les logs Jenkins complets
2. Exécuter le build Docker manuellement sur l'agent
3. Vérifier la configuration réseau Docker
4. S'assurer que le firewall n'interfère pas
