// Pipeline déclaratif : construit les 3 images du projet, les tague
// et les pousse sur Docker Hub.
pipeline {

    // agent any : le build s'exécute sur le nœud Jenkins lui-même (le conteneur
    // "jenkins"), qui possède le client Docker et accède au moteur de Docker
    // Desktop via le socket monté.
    agent any

    environment {
        // Compte Docker Hub : valeur non secrète, utilisée pour nommer les images.
        DOCKERHUB_USER = 'mamoudoubarry'
        // Tag principal = numéro de build Jenkins : chaque build donne une version unique.
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        // ---------------------------------------------------------------
        // Stage 1 - Checkout : récupère le code depuis le dépôt Git
        // configuré dans le job (Pipeline script from SCM).
        // ---------------------------------------------------------------
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ---------------------------------------------------------------
        // Stage 2 - Build : une image par microservice, à partir du
        // Dockerfile de chaque dossier. Tag = numéro de build.
        // ---------------------------------------------------------------
        stage('Build') {
            steps {
                sh 'docker build -t $DOCKERHUB_USER/bijouterie-frontend:$IMAGE_TAG ./frontend'
                sh 'docker build -t $DOCKERHUB_USER/bijouterie-backend:$IMAGE_TAG ./backend'
                sh 'docker build -t $DOCKERHUB_USER/bijouterie-mysql:$IMAGE_TAG ./database'
            }
        }

        // ---------------------------------------------------------------
        // Stage 3 - Tag : ajoute un second tag "latest" à chaque image
        // (double tag : numéro de build + latest).
        // ---------------------------------------------------------------
        stage('Tag') {
            steps {
                sh 'docker tag $DOCKERHUB_USER/bijouterie-frontend:$IMAGE_TAG $DOCKERHUB_USER/bijouterie-frontend:latest'
                sh 'docker tag $DOCKERHUB_USER/bijouterie-backend:$IMAGE_TAG $DOCKERHUB_USER/bijouterie-backend:latest'
                sh 'docker tag $DOCKERHUB_USER/bijouterie-mysql:$IMAGE_TAG $DOCKERHUB_USER/bijouterie-mysql:latest'
            }
        }

        // ---------------------------------------------------------------
        // Stage 4 - Push : connexion à Docker Hub avec le credential
        // "dockerhub-credentials", puis envoi des 3 images avec leurs 2 tags.
        // withCredentials expose le token dans des variables masquées
        // dans les logs, il n'est jamais écrit dans le Jenkinsfile.
        // ---------------------------------------------------------------
        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN')]) {
                    sh 'echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-frontend:$IMAGE_TAG'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-frontend:latest'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-backend:$IMAGE_TAG'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-backend:latest'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-mysql:$IMAGE_TAG'
                    sh 'docker push $DOCKERHUB_USER/bijouterie-mysql:latest'
                }
            }
        }
    }

    // post : s'exécute à la fin, que le build réussisse ou non.
    post {
        always {
            // Supprime la session Docker Hub du conteneur Jenkins.
            sh 'docker logout || true'
        }
    }
}