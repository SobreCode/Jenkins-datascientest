pipeline {
    agent any

    environment { 
        DOCKER_ID = "dstdockerhub"
        DOCKER_IMAGE = "datascientestapi"
        // Utilisation de env.BUILD_ID pour avoir un tag unique à chaque exécution
        DOCKER_TAG = "v.${env.BUILD_ID}.0"
    }

    stages {
        stage('Build & Test') {
            steps {
                // On utilise un conteneur Python pour installer les dépendances et tester
                // On monte le dossier courant $(pwd) dans /app pour que le conteneur voie le code
                sh """
                docker run --rm -v "\$(pwd):/app" -w /app python:3.9-slim bash -c "
                    if [ -f requirements.txt ]; then
                        pip install --no-cache-dir -r requirements.txt
                    fi
                    python -m unittest discover
                "
                """
            }
        }

        stage ('Deploying') {
            steps {
                script {
                    // On construit l'image finale et on lance le conteneur sur le serveur
                    sh """
                    docker build -t ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
                    
                    # Nettoyage de l'ancien conteneur s'il existe
                    docker rm -f jenkins || true 
                    
                    # Lancement du nouveau conteneur
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
                echo "Approbation reçue, passage à l'étape de Pushing."
            }
        }

        stage ('Pushing and merging') {
            parallel {
                stage ('Pushing') {
                    steps {
                        // Utilise l'ID de credentials 'docker_jenkins' configuré dans l'interface Jenkins
                        withCredentials([usernamePassword(credentialsId: 'docker_jenkins', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                            sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                            sh "docker push ${env.DOCKER_ID}/${env.DOCKER_IMAGE}:${env.env.DOCKER_TAG}"
                        }
                    }
                }
                stage ('Merging') {
                    steps {
                        // Logique de merge Git si nécessaire
                        echo 'Merging simulé terminé.'
                    }
                }
            }
        }
    }

    post {
        always {
            // Déconnexion de Docker Hub par sécurité
            sh 'docker logout || true'
        }
        success {
            echo "Le pipeline s'est terminé avec succès !"
        }
        failure {
            echo "Le pipeline a échoué. Vérifiez les logs ci-dessus."
        }
    }
}