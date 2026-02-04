pipeline {
    agent any
    
    environment { 
        DOCKER_ID = "dstdockerhub"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${env.BUILD_ID}.0"
    }

    stages {
        stage('Build & Test') {
            steps {
                // On lance un conteneur python juste pour exécuter les tests
                // Le -v ${WORKSPACE}:/app permet au conteneur de voir votre code
                sh """
                docker run --rm -v ${WORKSPACE}:/app -w /app python:3.9-slim bash -c "
                    pip install --no-cache-dir -r requirements.txt && \
                    python -m unittest discover
                "
                """
            }
        }

        stage ('Deploying') {
            steps {
                script {
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
                        // Utilisation de withCredentials pour Docker Hub
                        withCredentials([usernamePassword(credentialsId: 'docker_jenkins', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                            sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                            sh "docker push ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        }
                    }
                }
                stage ('Merging') {
                    steps {
                        echo 'Merging skipped'
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker logout || true'
        }
    }
}