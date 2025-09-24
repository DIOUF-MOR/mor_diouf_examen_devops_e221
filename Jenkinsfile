pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_REPO = 'daaroukhoudos04/mor_diouf_examen_devops_e221'
        RENDER_API_KEY = credentials('render-api-key')
        RENDER_SERVICE_ID = 'srv-d38he56r433s73fi5h20'
    }

    tools {
        maven 'Maven-3.8'
        jdk 'JDK-11'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [
                        [$class: 'CloneOption', depth: 1, shallow: true]
                    ],
                    userRemoteConfigs: [[
                        credentialsId: 'github-token',
                        url: 'https://github.com/DIOUF-MOR/mor_diouf_examen_devops_e221.git'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} .
                        docker tag ${DOCKERHUB_REPO}:${BUILD_NUMBER} ${DOCKERHUB_REPO}:latest
                    """
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    sh """
                        echo "Tentative de connexion à DockerHub..."
                        echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin

                        echo " Push  de l'image ${DOCKERHUB_REPO}:${BUILD_NUMBER}..."
                        docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}

                        echo "Push de l'image ${DOCKERHUB_REPO}:latest..."
                        docker push ${DOCKERHUB_REPO}:latest

                        echo "✅ Images poussées avec succès vers DockerHub"
                    """
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                script {
                    sh """
                        echo "🚀 Déclenchement du déploiement Render..."
                        curl -X POST https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys \\
                        -H "Authorization: Bearer \${RENDER_API_KEY}" \\
                        -H "Content-Type: application/json" \\
                        -d '{"clearCache": "clear"}' \\
                        -w "HTTP Status: %{http_code}\\n"
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
            echo "🔗 Vérifiez votre image sur: https://hub.docker.com/r/${DOCKERHUB_REPO}"
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
        always {
            deleteDir()
        }
    }
}