# Configuration Jenkins pour Spray_Info

## Prérequis sur le serveur Jenkins

### 1. Installer Node.js
\`\`\`bash
# Sur Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Vérifier l'installation
node --version
npm --version
\`\`\`

### 2. Installer Docker
\`\`\`bash
# Sur Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y docker.io

# Ajouter l'utilisateur Jenkins au groupe docker
sudo usermod -aG docker jenkins

# Redémarrer Jenkins
sudo systemctl restart jenkins
\`\`\`

### 3. Plugins Jenkins requis
Installer ces plugins depuis Jenkins > Manage Jenkins > Manage Plugins :
- **Git Plugin** - Pour cloner le repository
- **Docker Pipeline** - Pour les commandes Docker
- **Pipeline** - Pour les pipelines
- **Workspace Cleanup** - Pour nettoyer l'espace de travail

## Configuration du Job Jenkins

### Étape 1 : Créer un nouveau Pipeline
1. Aller dans Jenkins Dashboard
2. Cliquer sur "New Item"
3. Entrer le nom : `spray-info-pipeline`
4. Sélectionner "Pipeline"
5. Cliquer "OK"

### Étape 2 : Configurer le Pipeline
Dans la configuration du job :

#### Section "General"
- ✅ Cocher "GitHub project" (optionnel)
- URL du projet : `https://github.com/votre-username/spray-info`

#### Section "Build Triggers"
- ✅ Cocher "Poll SCM" pour vérifier les changements
- Schedule : `H/5 * * * *` (vérifie toutes les 5 minutes)

OU

- ✅ Cocher "GitHub hook trigger for GITScm polling" (si webhook configuré)

#### Section "Pipeline"
- **Definition** : Pipeline script from SCM
- **SCM** : Git
- **Repository URL** : URL de votre repository
- **Credentials** : Ajouter vos credentials Git si nécessaire
- **Branch Specifier** : `*/main` ou `*/master`
- **Script Path** : `Jenkinsfile`

### Étape 3 : Sauvegarder et Tester
1. Cliquer "Save"
2. Cliquer "Build Now"
3. Observer les logs dans "Console Output"

## Résolution des problèmes courants

### Problème : "docker: command not found"
**Solution** :
\`\`\`bash
# Vérifier que Docker est installé
docker --version

# Vérifier que Jenkins peut utiliser Docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
\`\`\`

### Problème : "npm: command not found"
**Solution** :
\`\`\`bash
# Installer Node.js sur le serveur Jenkins
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
\`\`\`

### Problème : "Permission denied" pour Docker
**Solution** :
\`\`\`bash
# Ajouter Jenkins au groupe docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# Vérifier les permissions
sudo chmod 666 /var/run/docker.sock
\`\`\`

### Problème : Port 3000 déjà utilisé
**Solution** :
\`\`\`bash
# Trouver et arrêter le processus
sudo lsof -i :3000
sudo kill -9 <PID>

# Ou changer le port dans le Jenkinsfile
# Modifier : -p 3002:3000
\`\`\`

### Problème : Build échoue à "npm run build"
**Solution** :
\`\`\`bash
# Vérifier les logs
# Souvent lié à des erreurs TypeScript ou ESLint

# Tester localement
npm install
npm run build
\`\`\`

## Test manuel du pipeline

### 1. Tester localement avant Jenkins
\`\`\`bash
# Cloner le projet
git clone <votre-repo>
cd spray-info

# Installer les dépendances
npm install

# Build l'application
npm run build

# Build l'image Docker
docker build -t spray-info:test .

# Tester l'image
docker run -d --name test -p 3000:3000 spray-info:test
curl http://localhost:3000

# Nettoyer
docker stop test
docker rm test
\`\`\`

### 2. Vérifier les logs Jenkins
- Aller dans le job
- Cliquer sur le dernier build
- Cliquer "Console Output"
- Chercher les erreurs en rouge

## Déploiement en production

### Option 1 : Déploiement local
Le pipeline déploie automatiquement sur `http://localhost:3000`

### Option 2 : Déploiement sur serveur distant
Modifier le stage "Deploy" dans le Jenkinsfile :
\`\`\`groovy
stage('Deploy') {
    steps {
        sh '''
            # Copier l'image vers le serveur
            docker save spray-info:${DOCKER_TAG} | ssh user@server docker load
            
            # Déployer sur le serveur
            ssh user@server "docker stop spray-info-app || true"
            ssh user@server "docker rm spray-info-app || true"
            ssh user@server "docker run -d --name spray-info-app -p 80:3000 spray-info:${DOCKER_TAG}"
        '''
    }
}
\`\`\`

## Monitoring

### Vérifier l'application
\`\`\`bash
# Vérifier que le conteneur tourne
docker ps | grep spray-info

# Voir les logs
docker logs spray-info-app

# Tester l'application
curl http://localhost:3000
\`\`\`

### Vérifier Jenkins
- Dashboard : Voir l'historique des builds
- Blue Ocean : Interface moderne pour visualiser les pipelines
- Logs : Console Output de chaque build

## Maintenance

### Nettoyer les anciennes images
\`\`\`bash
# Supprimer les images non utilisées
docker image prune -a -f

# Supprimer les conteneurs arrêtés
docker container prune -f
\`\`\`

### Sauvegarder la configuration
\`\`\`bash
# Sauvegarder le Jenkinsfile dans Git
git add Jenkinsfile
git commit -m "Update Jenkins pipeline"
git push
