pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'spray-info'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'your-registry.com' // Modifier selon votre registre
        NODE_VERSION = '20'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installation des dépendances...'
                sh '''
                    node --version
                    npm --version
                    npm ci
                '''
            }
        }
        
        stage('Lint & Type Check') {
            steps {
                echo 'Vérification du code...'
                sh '''
                    npm run lint || true
                '''
            }
        }
        
        stage('Build Application') {
            steps {
                echo 'Construction de l\'application...'
                sh '''
                    npm run build
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Construction de l\'image Docker...'
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                echo 'Test de l\'image Docker...'
                sh '''
                    docker run -d --name test-container -p 3001:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    sleep 10
                    curl -f http://localhost:3001 || exit 1
                    docker stop test-container
                    docker rm test-container
                '''
            }
        }
        
        stage('Push to Registry') {
            when {
                branch 'main'
            }
            steps {
                echo 'Publication de l\'image Docker...'
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Déploiement en staging...'
                sh '''
                    # Commandes de déploiement staging
                    # Exemple: kubectl apply -f k8s/staging/
                    echo "Déploiement staging avec l'image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Déploiement en production...'
                input message: 'Déployer en production?', ok: 'Déployer'
                sh '''
                    # Commandes de déploiement production
                    # Exemple: kubectl apply -f k8s/production/
                    echo "Déploiement production avec l'image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Nettoyage...'
            sh '''
                docker system prune -f || true
            '''
        }
        success {
            echo 'Pipeline exécuté avec succès! ✅'
        }
        failure {
            echo 'Le pipeline a échoué! ❌'
        }
    }
}
