# Guide de Déploiement - Spray Info

## Configuration du Déploiement

### Variables d'Environnement Jenkins

Configurez ces variables dans Jenkins pour personnaliser le déploiement:

\`\`\`bash
DEPLOY_PORT=3000          # Port sur lequel l'application sera accessible
NODE_ENV=production       # Environnement d'exécution
\`\`\`

### Configuration dans Jenkins

1. **Aller dans votre job Jenkins**
2. **Cliquer sur "Configure"**
3. **Dans "Build Environment", cocher "Inject environment variables"**
4. **Ajouter les variables:**
   \`\`\`
   DEPLOY_PORT=3000
   \`\`\`

## Fonctionnalités du Stage Deploy

### 1. Déploiement avec Rollback Automatique

Le stage Deploy inclut maintenant:
- ✅ Sauvegarde automatique de l'ancien conteneur
- ✅ Health check après déploiement
- ✅ Rollback automatique en cas d'échec
- ✅ Vérification de disponibilité de l'application

### 2. Processus de Déploiement

\`\`\`
1. Sauvegarde de l'ancien conteneur (spray-info-app → spray-info-app-old)
2. Arrêt de l'ancien conteneur
3. Démarrage du nouveau conteneur
4. Attente de 10 secondes
5. Health check (5 tentatives avec 5 secondes d'intervalle)
6. Si succès: Suppression de l'ancien conteneur
7. Si échec: Rollback vers l'ancienne version
\`\`\`

### 3. Ports Configurables

Par défaut, l'application est déployée sur le port 3000. Pour changer:

\`\`\`bash
# Dans Jenkins, définir la variable d'environnement
DEPLOY_PORT=8080
\`\`\`

## Déploiement Manuel

### Déploiement Simple

\`\`\`bash
# Arrêter l'ancien conteneur
docker stop spray-info-app
docker rm spray-info-app

# Démarrer le nouveau
docker run -d \
  --name spray-info-app \
  -p 3000:3000 \
  --restart unless-stopped \
  spray-info:latest

# Vérifier
curl http://localhost:3000
\`\`\`

### Déploiement avec Docker Compose

\`\`\`bash
# Utiliser le fichier docker-compose.yml
docker-compose up -d

# Vérifier les logs
docker-compose logs -f

# Arrêter
docker-compose down
\`\`\`

## Résolution des Problèmes

### Le conteneur ne démarre pas

\`\`\`bash
# Vérifier les logs
docker logs spray-info-app

# Vérifier si le port est déjà utilisé
netstat -tulpn | grep 3000
# ou
lsof -i :3000

# Changer le port si nécessaire
docker run -d --name spray-info-app -p 8080:3000 spray-info:latest
\`\`\`

### L'application ne répond pas

\`\`\`bash
# Vérifier que le conteneur est en cours d'exécution
docker ps | grep spray-info-app

# Vérifier les logs en temps réel
docker logs -f spray-info-app

# Redémarrer le conteneur
docker restart spray-info-app

# Tester la connexion
curl -v http://localhost:3000
\`\`\`

### Rollback Manuel

\`\`\`bash
# Lister les images disponibles
docker images | grep spray-info

# Arrêter le conteneur actuel
docker stop spray-info-app
docker rm spray-info-app

# Démarrer une version précédente
docker run -d \
  --name spray-info-app \
  -p 3000:3000 \
  --restart unless-stopped \
  spray-info:123  # Remplacer 123 par le numéro de build précédent
\`\`\`

### Nettoyage des Anciennes Images

\`\`\`bash
# Voir toutes les images
docker images | grep spray-info

# Supprimer les anciennes images (garder les 3 dernières)
docker images | grep spray-info | tail -n +4 | awk '{print $3}' | xargs docker rmi

# Nettoyer toutes les images non utilisées
docker image prune -a
\`\`\`

## Déploiement Multi-Environnement

### Staging

\`\`\`bash
docker run -d \
  --name spray-info-staging \
  -p 3001:3000 \
  -e NODE_ENV=staging \
  spray-info:latest
\`\`\`

### Production

\`\`\`bash
docker run -d \
  --name spray-info-production \
  -p 3000:3000 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  spray-info:latest
\`\`\`

## Monitoring

### Vérifier l'état de l'application

\`\`\`bash
# Statut du conteneur
docker ps | grep spray-info-app

# Utilisation des ressources
docker stats spray-info-app

# Logs en temps réel
docker logs -f spray-info-app

# Health check
curl -f http://localhost:3000 && echo "OK" || echo "FAIL"
\`\`\`

### Logs

\`\`\`bash
# Derniers logs
docker logs --tail 100 spray-info-app

# Logs depuis une date
docker logs --since 2024-01-01T00:00:00 spray-info-app

# Suivre les logs
docker logs -f spray-info-app
\`\`\`

## Sécurité

### Bonnes Pratiques

1. **Ne pas exposer de ports inutiles**
2. **Utiliser des variables d'environnement pour les secrets**
3. **Mettre à jour régulièrement les images de base**
4. **Limiter les ressources du conteneur**

\`\`\`bash
# Exemple avec limites de ressources
docker run -d \
  --name spray-info-app \
  -p 3000:3000 \
  --memory="512m" \
  --cpus="1.0" \
  --restart unless-stopped \
  spray-info:latest
\`\`\`

## Support

En cas de problème persistant:

1. Vérifier les logs Jenkins
2. Vérifier les logs Docker
3. Vérifier la configuration réseau
4. Vérifier l'espace disque disponible
5. Consulter la documentation Docker
