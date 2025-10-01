pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'spray-info'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        NODE_VERSION = '20'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Récupération du code source...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '📥 Installation des dépendances...'
                sh 'npm ci --prefer-offline --no-audit'
            }
        }
        
        stage('Lint') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '🔍 Vérification du code...'
                sh 'npm run lint || echo "⚠️ Lint warnings trouvés"'
            }
        }
        
        stage('Build Application') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '🏗️ Construction de l\'application...'
                sh 'npm run build'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐳 Construction de l\'image Docker...'
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                echo '🧪 Test de l\'image Docker...'
                script {
                    sh """
                        # Arrêter et supprimer le conteneur s'il existe
                        docker stop test-spray-info || true
                        docker rm test-spray-info || true
                        
                        # Démarrer le conteneur de test
                        docker run -d --name test-spray-info -p 3001:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        # Attendre que l'application démarre
                        echo "⏳ Attente du démarrage de l'application..."
                        sleep 10
                        
                        # Vérifier les logs
                        docker logs test-spray-info
                        
                        # Tester l'application
                        curl -f http://localhost:3001 || (docker logs test-spray-info && exit 1)
                        
                        echo "✅ Test réussi!"
                        
                        # Nettoyer
                        docker stop test-spray-info
                        docker rm test-spray-info
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo '🚀 Déploiement...'
                script {
                    sh """
                        echo "Déploiement de l'image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        
                        # Arrêter l'ancien conteneur
                        docker stop spray-info-app || true
                        docker rm spray-info-app || true
                        
                        # Démarrer le nouveau conteneur
                        docker run -d \
                            --name spray-info-app \
                            -p 3000:3000 \
                            --restart unless-stopped \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        echo "✅ Application déployée sur http://localhost:3000"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo '🧹 Nettoyage...'
            sh '''
                # Nettoyer les conteneurs de test
                docker stop test-spray-info || true
                docker rm test-spray-info || true
                
                # Nettoyer les images non utilisées
                docker image prune -f || true
            '''
        }
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué!'
            sh '''
                # Afficher les logs en cas d'échec
                docker logs test-spray-info || true
                docker logs spray-info-app || true
            '''
        }
    }
}
