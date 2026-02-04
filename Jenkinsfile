pipeline {
  agent any
  environment { 
    DOCKER_ID = "dstdockerhub"
    DOCKER_IMAGE = "datascientestapi"
    DOCKER_TAG = "v.${env.BUILD_ID}.0" // Utiliser env.BUILD_ID est plus sûr
  }
  stages {
    stage ('Building') {
        steps {
            sh 'pip install -r requirements.txt'
        }
    }
    stage ('Testing') {
        steps {
            sh 'python -m unittest'
        }
    }
    stage ('Deploying') {
        steps {
            script {
                sh """
                # On construit l'image
                docker build -t ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
                
                # On nettoie
                docker rm -f jenkins || true 
                
                # Correction ici : ajout de env. pour chaque variable
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
            echo "Déploiement approuvé par l'utilisateur."
        }
    }
    stage ('Pushing and merging') {
        parallel {
            stage ('Pushing') {
                environment {
                    DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')
                }
                steps {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
            stage ('Merging') {
                steps {
                    echo 'Merging done'
                }
            }
        }
    }
  }
  post {
    always {
        sh 'docker logout'
    }
  }
}