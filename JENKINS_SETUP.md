# Configuration Jenkins pour Spray_Info

## Prérequis

### 1. Plugins Jenkins nécessaires
Installez ces plugins via Jenkins > Manage Jenkins > Manage Plugins:
- Docker Pipeline
- Docker Commons
- Git
- Pipeline
- Credentials Binding

### 2. Configuration Docker dans Jenkins

#### Option A: Jenkins avec Docker installé sur l'hôte
\`\`\`bash
# Ajouter l'utilisateur jenkins au groupe docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
\`\`\`

#### Option B: Jenkins dans Docker (Docker-in-Docker)
\`\`\`bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
\`\`\`

### 3. Configuration des Credentials

#### Docker Hub Credentials
1. Allez dans Jenkins > Manage Jenkins > Manage Credentials
2. Cliquez sur "(global)" puis "Add Credentials"
3. Sélectionnez "Username with password"
4. ID: `dockerhub-credentials`
5. Username: votre username Docker Hub
6. Password: votre token Docker Hub
7. Cliquez sur "Create"

### 4. Configuration du Pipeline

1. Créez un nouveau Pipeline Job dans Jenkins
2. Dans "Pipeline" section, sélectionnez "Pipeline script from SCM"
3. SCM: Git
4. Repository URL: votre URL Git
5. Branch: */main (ou */master)
6. Script Path: Jenkinsfile

## Structure des Branches

- `main` ou `master`: Production (déploiement avec validation manuelle)
- `develop`: Staging (déploiement automatique)
- `feature/*`: Branches de développement (build et test uniquement)

## Variables d'Environnement à Configurer

Dans Jenkins > Configure > Environment Variables:

\`\`\`
DOCKER_REGISTRY=docker.io
DOCKER_IMAGE=votre-username/spray-info
\`\`\`

## Commandes de Déploiement

### Déploiement Local avec Docker Compose
\`\`\`bash
# Staging
docker-compose -f docker-compose.yml up -d

# Production
docker-compose -f docker-compose.yml up -d
\`\`\`

### Déploiement Kubernetes (optionnel)
\`\`\`bash
# Créer les fichiers k8s/deployment.yml et k8s/service.yml
kubectl apply -f k8s/
\`\`\`

## Troubleshooting

### Erreur: "Cannot connect to Docker daemon"
\`\`\`bash
# Vérifier que Docker est accessible
docker ps

# Redémarrer Jenkins
sudo systemctl restart jenkins
\`\`\`

### Erreur: "npm: command not found"
Le pipeline utilise maintenant un agent Docker avec Node.js préinstallé.

### Erreur: "Permission denied" sur Docker
\`\`\`bash
sudo chmod 666 /var/run/docker.sock
# ou
sudo usermod -aG docker $USER
\`\`\`

### Port déjà utilisé lors des tests
Le pipeline utilise maintenant un port dynamique basé sur BUILD_NUMBER.

## Notifications (Optionnel)

Pour activer les notifications par email en cas d'échec:
1. Installez le plugin "Email Extension"
2. Configurez SMTP dans Jenkins > Configure System
3. Décommentez la section emailext dans le Jenkinsfile

## Monitoring

Vérifiez les logs du build:
\`\`\`bash
# Logs Jenkins
tail -f /var/jenkins_home/jobs/spray-info/builds/lastBuild/log

# Logs Docker
docker logs spray-info
