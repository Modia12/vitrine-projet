# Guide de Dépannage Jenkins

## Problème: "Install Dependencies" échoue

### Solution 1: Installer Node.js sur l'agent Jenkins

\`\`\`bash
# Sur Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Sur CentOS/RHEL
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs

# Vérifier l'installation
node --version
npm --version
\`\`\`

### Solution 2: Activer Docker dans Jenkins

1. **Installer le plugin Docker Pipeline**
   - Aller dans Jenkins → Manage Jenkins → Manage Plugins
   - Onglet "Available"
   - Chercher "Docker Pipeline"
   - Installer et redémarrer Jenkins

2. **Donner accès Docker à l'utilisateur Jenkins**
   \`\`\`bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   \`\`\`

3. **Vérifier que Docker fonctionne**
   \`\`\`bash
   sudo -u jenkins docker ps
   \`\`\`

### Solution 3: Utiliser un agent Jenkins avec Node.js

Dans votre configuration Jenkins, créer un agent avec Node.js préinstallé:

1. Manage Jenkins → Manage Nodes and Clouds
2. New Node
3. Configurer avec Node.js installé

### Solution 4: Jenkinsfile simplifié (sans Docker)

Si vous ne pouvez pas utiliser Docker, le nouveau Jenkinsfile détecte automatiquement Node.js local ou installe nvm.

## Vérification rapide

Exécutez ces commandes sur votre agent Jenkins:

\`\`\`bash
# Vérifier Node.js
node --version

# Vérifier npm
npm --version

# Vérifier Docker
docker --version

# Vérifier les permissions Docker
docker ps
\`\`\`

## Logs utiles

\`\`\`bash
# Logs Jenkins
sudo tail -f /var/log/jenkins/jenkins.log

# Logs Docker
docker logs <container-name>

# Vérifier l'utilisateur Jenkins
sudo -u jenkins whoami
sudo -u jenkins groups
\`\`\`

## Erreurs courantes

### "npm: command not found"
- Node.js n'est pas installé → Voir Solution 1

### "docker: command not found"
- Docker n'est pas installé ou pas dans le PATH

### "permission denied while trying to connect to Docker"
- L'utilisateur jenkins n'a pas accès à Docker → Voir Solution 2

### "Cannot connect to the Docker daemon"
- Le service Docker n'est pas démarré:
  \`\`\`bash
  sudo systemctl start docker
  sudo systemctl enable docker
  \`\`\`

## Configuration recommandée

Pour un environnement de production stable:

1. **Installer Node.js directement sur l'agent Jenkins** (Solution 1)
2. **Garder Docker pour les images de production uniquement**
3. **Utiliser un cache npm pour accélérer les builds**

\`\`\`groovy
// Dans Jenkinsfile, ajouter:
environment {
    NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
}
