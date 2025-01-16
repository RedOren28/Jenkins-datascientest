pipeline {
    agent any
    environment { 
        DOCKER_ID = "redoren"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0" 
    }
    stages {
        stage('Building') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Testing') {
            steps {
                sh 'python3 --version'  // Vérifier la version de Python3
                sh 'pip3 --version'     // Vérifier la version de pip
                sh 'python3 -m unittest'
            }
        }
        stage('Deploying') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('User Acceptance') {
            steps {
                script {
                    // input(
                    //     message: "Proceed to push to main?", 
                    //     ok: "Yes"
                    // )
                    echo "Automatically proceeding with push."
                }
            }
        }
        stage('Pushing and Merging') {
            parallel {
                stage('Pushing Image') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
                    }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'docker_jenkins', passwordVariable: 'DOCKERHUB_PSW', usernameVariable: 'DOCKERHUB_USR')]) {
                                // Connexion à Docker Hub avec les informations d'identification injectées
                                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                                // Pousser l'image Docker
                                sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                            }
                        }
                    }
                }
                stage('Merging') {
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
