pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_REPO = 'DIOUF-MOR/mor_diouf_examen_devops_e221'
        RENDER_API_KEY = credentials('render-api-key')
        RENDER_SERVICE_ID = 'srv-d3808q3e5dus739ooksg'
    }

    tools {
        maven 'Maven-3.8' // Assurez-vous que Maven est configuré dans Jenkins
        jdk 'JDK-11'      // Assurez-vous que JDK 11 est configuré
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/DIOUF-MOR/mor_diouf_examen_devops_e221.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} ."
                    sh "docker tag ${DOCKERHUB_REPO}:${BUILD_NUMBER} ${DOCKERHUB_REPO}:latest"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh "docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKERHUB_REPO}:latest"
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
                        curl -X POST https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys \
                        -H "Authorization: Bearer ${RENDER_API_KEY}" \
                        -H "Content-Type: application/json" \
                        -d '{
                            "clearCache": "clear"
                        }'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
        always {
            cleanWs()
        }
    }
}
