pipeline {
    agent any
    
    environment { 
        DOCKER_ID = "dstdockerhub"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${env.BUILD_ID}.0"
    }

    stages {
        // Cette étape utilise une image Docker Python pour éviter l'erreur "pip3 not found"
        stage('Build & Test') {
            agent {
                docker { 
                    image 'python:3.9-slim'
                    // On réutilise le cache pip pour aller plus vite
                    args '-v $HOME/.cache/pip:/root/.cache/pip'
                }
            }
            steps {
                sh 'pip install --user -r requirements.txt'
                sh 'python -m unittest discover' 
            }
        }

        stage ('Deploying') {
            steps {
                script {
                    // Utilisation de env. partout pour éviter les erreurs Groovy
                    sh """
                    docker build -t ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
                    docker rm -f jenkins || true 
                    docker run -d -p 8000:8000 --name jenkins ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    """
                }
            }
        }

        stage ('User acceptance') {
            input {
                message "Souhaitez-vous déployer le code sur la branche main ?"
                ok "Oui"
            }
            steps {
                echo "Approbation reçue."
            }
        }

        stage ('Pushing and merging') {
            parallel {
                stage ('Pushing') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'docker_jenkins', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                            sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                            sh "docker push ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        }
                    }
                }
                stage ('Merging') {
                    steps {
                        echo 'Merging logic would go here'
                    }
                }
            }
        }
    }
    
    post {
        always {
            // On ne logout que si on était loggé
            sh 'docker logout || true'
        }
    }
}