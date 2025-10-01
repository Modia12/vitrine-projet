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
        
        stage('Check Prerequisites') {
            steps {
                script {
                    echo '🔍 Vérification des prérequis...'
                    
                    // Vérifier si Docker est disponible
                    def dockerAvailable = sh(script: 'command -v docker', returnStatus: true) == 0
                    env.DOCKER_AVAILABLE = dockerAvailable.toString()
                    
                    if (dockerAvailable) {
                        echo '✅ Docker est disponible'
                    } else {
                        echo '⚠️ Docker n\'est pas disponible'
                    }
                    
                    // Vérifier si Node.js est disponible
                    def nodeAvailable = sh(script: 'command -v node', returnStatus: true) == 0
                    env.NODE_AVAILABLE = nodeAvailable.toString()
                    
                    if (nodeAvailable) {
                        def nodeVersion = sh(script: 'node --version', returnStdout: true).trim()
                        echo "✅ Node.js est disponible: ${nodeVersion}"
                    } else {
                        echo '⚠️ Node.js n\'est pas disponible localement'
                    }
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    echo '📥 Installation des dépendances...'
                    
                    // Si Node.js est disponible localement, l'utiliser directement
                    if (env.NODE_AVAILABLE == 'true') {
                        echo '🟢 Utilisation de Node.js local'
                        sh '''
                            node --version
                            npm --version
                            npm ci --prefer-offline --no-audit || npm install
                        '''
                    }
                    // Sinon, utiliser Docker si disponible
                    else if (env.DOCKER_AVAILABLE == 'true') {
                        echo '🐳 Utilisation de Docker avec Node.js'
                        sh '''
                            docker run --rm \
                                -v "$(pwd)":/app \
                                -w /app \
                                node:20-alpine \
                                sh -c "npm ci --prefer-offline --no-audit || npm install"
                        '''
                    }
                    // Sinon, installer Node.js via nvm
                    else {
                        echo '📦 Installation de Node.js via nvm'
                        sh '''
                            # Installer nvm si nécessaire
                            if [ ! -d "$HOME/.nvm" ]; then
                                curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                            fi
                            
                            # Charger nvm
                            export NVM_DIR="$HOME/.nvm"
                            [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                            
                            # Installer et utiliser Node.js 20
                            nvm install 20
                            nvm use 20
                            
                            # Installer les dépendances
                            npm ci --prefer-offline --no-audit || npm install
                        '''
                    }
                    
                    echo '✅ Dépendances installées avec succès'
                }
            }
        }
        
        stage('Lint') {
            steps {
                script {
                    echo '🔍 Vérification du code...'
                    
                    if (env.NODE_AVAILABLE == 'true') {
                        sh 'npm run lint || echo "⚠️ Lint warnings trouvés"'
                    } else if (env.DOCKER_AVAILABLE == 'true') {
                        sh '''
                            docker run --rm \
                                -v "$(pwd)":/app \
                                -w /app \
                                node:20-alpine \
                                sh -c "npm run lint || echo '⚠️ Lint warnings trouvés'"
                        '''
                    } else {
                        sh '''
                            export NVM_DIR="$HOME/.nvm"
                            [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                            nvm use 20
                            npm run lint || echo "⚠️ Lint warnings trouvés"
                        '''
                    }
                }
            }
        }
        
        stage('Build Application') {
            steps {
                script {
                    echo '🏗️ Construction de l\'application...'
                    
                    if (env.NODE_AVAILABLE == 'true') {
                        sh 'npm run build'
                    } else if (env.DOCKER_AVAILABLE == 'true') {
                        sh '''
                            docker run --rm \
                                -v "$(pwd)":/app \
                                -w /app \
                                node:20-alpine \
                                npm run build
                        '''
                    } else {
                        sh '''
                            export NVM_DIR="$HOME/.nvm"
                            [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                            nvm use 20
                            npm run build
                        '''
                    }
                    
                    echo '✅ Application construite avec succès'
                }
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
                docker stop test-spray-info 2>/dev/null || true
                docker rm test-spray-info 2>/dev/null || true
                
                # Nettoyer les images non utilisées
                docker image prune -f 2>/dev/null || true
            '''
        }
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué!'
            sh '''
                # Afficher les logs en cas d'échec
                docker logs test-spray-info 2>/dev/null || true
                docker logs spray-info-app 2>/dev/null || true
            '''
        }
    }
}
