# Spray_Info - Site Vitrine

Site vitrine professionnel pour Spray_Info, spécialisé dans les services de peinture par spray.

## 🚀 Démarrage rapide

### Développement local

\`\`\`bash
# Installation des dépendances
npm install

# Lancement du serveur de développement
npm run dev
\`\`\`

Ouvrez [http://localhost:3000](http://localhost:3000) dans votre navigateur.

## 🐳 Docker

### Construction de l'image

\`\`\`bash
docker build -t spray-info:latest .
\`\`\`

### Exécution du conteneur

\`\`\`bash
docker run -p 3000:3000 spray-info:latest
\`\`\`

## 🔧 Jenkins CI/CD

Le projet inclut un `Jenkinsfile` pour l'intégration et le déploiement continus.

### Prérequis Jenkins

- Jenkins avec Docker installé
- Plugin Docker Pipeline
- Credentials configurés pour le registre Docker

### Branches

- `main` : Production (déploiement manuel après validation)
- `develop` : Staging (déploiement automatique)

### Variables d'environnement à configurer

Dans Jenkins, configurez les variables suivantes :

- `DOCKER_REGISTRY` : URL de votre registre Docker
- Credentials `docker-credentials` pour l'authentification au registre

## 📦 Structure du projet

\`\`\`
spray-info/
├── app/
│   ├── page.tsx          # Page d'accueil
│   ├── layout.tsx        # Layout principal
│   └── globals.css       # Styles globaux
├── components/
│   └── ui/               # Composants UI réutilisables
├── Dockerfile            # Configuration Docker
├── Jenkinsfile           # Pipeline CI/CD
└── next.config.mjs       # Configuration Next.js
\`\`\`

## 🎨 Design

Le site utilise une palette de couleurs professionnelle :
- **Primaire** : Amber (#d97706) - Chaleur et professionnalisme
- **Secondaire** : Cyan (#0891b2) - Fraîcheur et modernité
- **Neutrals** : Blanc, gris clair, gris foncé

Typographie :
- **Titres** : Playfair Display (serif élégant)
- **Corps** : Source Sans Pro (sans-serif lisible)

## 🚢 Déploiement

### Vercel (Recommandé)

Le moyen le plus simple est de déployer via Vercel :

1. Cliquez sur "Publish" dans l'interface v0
2. Ou connectez votre repo GitHub à Vercel

### Docker + Kubernetes

Pour un déploiement sur infrastructure personnalisée :

\`\`\`bash
# Build
docker build -t spray-info:v1.0.0 .

# Push vers votre registre
docker tag spray-info:v1.0.0 your-registry.com/spray-info:v1.0.0
docker push your-registry.com/spray-info:v1.0.0

# Déploiement K8s (exemple)
kubectl apply -f k8s/
\`\`\`

## 📝 Licence

© 2025 Spray_Info. Tous droits réservés.
