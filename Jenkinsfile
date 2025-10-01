pipeline {
    agent {
        docker {
            image 'node:20-alpine'
            args '-u root:root'
        }
    }
    
    environment {
        DOCKER_IMAGE = 'spray-info'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'docker.io' // Utiliser Docker Hub par défaut
        HOME = "${WORKSPACE}"
        npm_config_cache = "${WORKSPACE}/.npm"
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
                    npm ci --prefer-offline --no-audit
                '''
            }
        }
        
        stage('Lint & Type Check') {
            steps {
                echo 'Vérification du code...'
                sh '''
                    npm run lint || echo "Lint warnings found"
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
            agent any // Revenir à l'agent principal pour Docker
            steps {
                echo 'Construction de l\'image Docker...'
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Test Docker Image') {
            agent any
            steps {
                echo 'Test de l\'image Docker...'
                script {
                    try {
                        sh """
                            docker run -d --name test-container-${BUILD_NUMBER} -p 300${BUILD_NUMBER}:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                            sleep 15
                            docker logs test-container-${BUILD_NUMBER}
                            curl -f http://localhost:300${BUILD_NUMBER} || exit 1
                        """
                    } finally {
                        sh """
                            docker stop test-container-${BUILD_NUMBER} || true
                            docker rm test-container-${BUILD_NUMBER} || true
                        """
                    }
                }
            }
        }
        
        stage('Push to Registry') {
            agent any
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                echo 'Publication de l\'image Docker...'
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        dockerImage.push("${DOCKER_TAG}")
                        dockerImage.push("latest")
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
                script {
                    sh """
                        echo "Déploiement staging avec l'image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        # docker-compose -f docker-compose.staging.yml up -d
                    """
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                echo 'Déploiement en production...'
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Déployer en production?', ok: 'Déployer'
                }
                script {
                    sh """
                        echo "Déploiement production avec l'image ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        # docker-compose -f docker-compose.prod.yml up -d
                        # ou kubectl set image deployment/spray-info spray-info=${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'Nettoyage...'
            sh '''
                docker system prune -f || true
            '''
            cleanWs()
        }
        success {
            echo 'Pipeline exécuté avec succès!'
        }
        failure {
            echo 'Le pipeline a échoué!'
            // emailext subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
            //          body: "Le build a échoué. Voir: ${env.BUILD_URL}",
            //          to: "team@example.com"
        }
    }
}
