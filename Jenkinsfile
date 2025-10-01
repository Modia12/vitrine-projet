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
            when {
                expression { env.DOCKER_AVAILABLE == 'true' }
            }
            steps {
                script {
                    echo '🐳 Construction de l\'image Docker...'
                    
                    try {
                        sh """
                            echo "📋 Vérification des fichiers nécessaires..."
                            ls -la
                            
                            echo "🔨 Construction de l'image Docker..."
                            docker build \
                                --no-cache \
                                --progress=plain \
                                -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                                -t ${DOCKER_IMAGE}:latest \
                                . 2>&1 | tee docker-build.log
                            
                            echo "✅ Image Docker construite avec succès"
                            docker images | grep ${DOCKER_IMAGE}
                        """
                    } catch (Exception e) {
                        echo "❌ Erreur lors de la construction de l'image Docker"
                        sh "cat docker-build.log || true"
                        throw e
                    }
                }
            }
        }
        
        stage('Test Docker Image') {
            when {
                expression { env.DOCKER_AVAILABLE == 'true' }
            }
            steps {
                script {
                    echo '🧪 Test de l\'image Docker...'
                    
                    try {
                        sh """
                            echo "🧹 Nettoyage des conteneurs existants..."
                            docker stop test-spray-info 2>/dev/null || true
                            docker rm test-spray-info 2>/dev/null || true
                            
                            echo "🚀 Démarrage du conteneur de test..."
                            docker run -d \
                                --name test-spray-info \
                                -p 3001:3000 \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                            
                            echo "⏳ Attente du démarrage (15 secondes)..."
                            sleep 15
                            
                            echo "📋 Vérification des logs du conteneur..."
                            docker logs test-spray-info
                            
                            echo "🔍 Test de l'application..."
                            for i in 1 2 3 4 5; do
                                if curl -f -s http://localhost:3001 > /dev/null; then
                                    echo "✅ Application répond correctement!"
                                    break
                                else
                                    echo "⏳ Tentative \$i/5..."
                                    sleep 3
                                fi
                            done
                            
                            # Vérifier une dernière fois
                            curl -f http://localhost:3001 || (
                                echo "❌ L'application ne répond pas"
                                docker logs test-spray-info
                                exit 1
                            )
                            
                            echo "🧹 Nettoyage du conteneur de test..."
                            docker stop test-spray-info
                            docker rm test-spray-info
                        """
                    } catch (Exception e) {
                        echo "❌ Erreur lors du test de l'image Docker"
                        sh "docker logs test-spray-info 2>/dev/null || true"
                        sh "docker stop test-spray-info 2>/dev/null || true"
                        sh "docker rm test-spray-info 2>/dev/null || true"
                        throw e
                    }
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
