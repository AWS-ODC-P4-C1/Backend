pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mormbathie/odc_backend:${BUILD_NUMBER}"
        GIT_CREDENTIALS_ID = 'github-creds_odc'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds_odc'
    }

    stages {
        stage('Clone code') {
            steps {
                git branch: 'main', credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/AWS-ODC-P4-C1/Backend.git'
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


        stage('Check Django App') {
            steps {
                dir('odc') {
                    sh 'python manage.py check'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}", "odc") // contexte = odc/
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    sh "docker stop odc_backend || true"
                    sh "docker rm odc_backend || true"
                    sh "docker run -d -p 8000:8000 --name odc_backend ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        success {
            mail to: 'mormbathie98@gmail.com',
                 subject: "Pipeline Jenkins réussi",
                 body: "L'application Django est déployée avec succès ! 🎉"
        }

        failure {
            mail to: 'mormbathie98@gmail.com',
                 subject: "Échec du pipeline Jenkins",
                 body: "Le pipeline a échoué. Vérifie Jenkins pour plus de détails."
        }
    }
}
