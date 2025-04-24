pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'tondockerhub/odc_backend'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Clone code') {
            steps {
                git 'https://github.com/TON_UTILISATEUR_GITHUB/TON_REPO.git'
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    docker.build("$DOCKER_IMAGE")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push $DOCKER_IMAGE"
                    }
                }
            }
        }

        stage('Deploy container') {
            steps {
                script {
                    // Stop old container (if exists)
                    sh "docker stop odc_backend || true"
                    sh "docker rm odc_backend || true"
                    // Run new container
                    sh "docker run -d -p 8000:8000 --name odc_backend $DOCKER_IMAGE"
                }
            }
        }
    }

    post {
        failure {
            mail to: 'tonmail@example.com',
                 subject: "Échec du pipeline Jenkins",
                 body: "Le pipeline a échoué. Vérifie Jenkins pour plus de détails."
        }
        success {
            mail to: 'tonmail@example.com',
                 subject: "Pipeline Jenkins réussi 🎉",
                 body: "Tout s'est bien passé. L'application est déployée !"
        }
    }
}
