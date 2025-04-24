pipeline {
    agent any // Utilise l'agent 'any' si Docker n'est pas une option

    environment {
        DOCKER_IMAGE = "mormbathie/odc_backend:${BUILD_NUMBER}" // Tag de l'image Docker
        DOCKER_CREDENTIALS_ID = 'github-creds_odc' // Identifiants DockerHub
    }

    stages {
        stage('Clone code') {
            steps {
                git branch: 'main', credentialsId: 'github-creds_odc', url: 'https://github.com/AWS-ODC-P4-C1/Backend.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('odc') {
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE ."
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
                    // Arrête et supprime l'ancien conteneur (si existe)
                    sh "docker stop odc_backend || true"
                    sh "docker rm odc_backend || true"

                    // Démarre le nouveau conteneur
                    sh "docker run -d -p 8000:8000 --name odc_backend $DOCKER_IMAGE"
                }
            }
        }
    }

    post {
        failure {
            mail to: 'mormbathie98@gmail.com',
                 subject: "❌ Échec du pipeline Jenkins",
                 body: "Le pipeline a échoué. Vérifie Jenkins pour plus de détails."
        }
        success {
            mail to: 'mormbathie98@gmail.com',
                 subject: "✅ Pipeline Jenkins réussi",
                 body: "Tout s'est bien passé. L'application est déployée ! 🎉"
        }
    }
}
