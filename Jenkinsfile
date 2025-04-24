pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mormbathie/odc_backend'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Clone code') {
            steps {
                git 'https://github.com/AWS-ODC-P4-C1/Backend.git'
            }
        }

        stage('Run tests') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python manage.py test'
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
                    sh "docker stop odc_backend || true"
                    sh "docker rm odc_backend || true"
                    sh "docker run -d -p 8000:8000 --name odc_backend $DOCKER_IMAGE"
                }
            }
        }
    }

    post {
        failure {
            mail to: 'mormbathie98@gmail.com',
                 subject: "Échec du pipeline Jenkins",
                 body: "Le pipeline a échoué. Vérifie Jenkins pour plus de détails."
        }
        success {
            mail to: 'mormbathie98@gmail.com',
                 subject: "Pipeline Jenkins réussi 🎉",
                 body: "Tout s'est bien passé. L'application est déployée !"
        }
    }
}
