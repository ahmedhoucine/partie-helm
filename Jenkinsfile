pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ahmedhoucine0/mon-app-helm'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Cloner le dépôt') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/ahmedhoucine/partie-helm.git'
            }
        }
        
        stage('Vérifier les fichiers') {
            steps {
                sh '''
                    echo "Liste des fichiers dans le dépôt:"
                    ls -la
                    echo "Contenu du répertoire:"
                    find . -type f -name "*.yaml" -o -name "*.yml" -o -name "Dockerfile" | head -20
                '''
            }
        }
        
        stage('Construire l\'image Docker') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
        
        stage('Pousser l\'image Docker') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    passwordVariable: 'DOCKER_PASSWORD', 
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh '''
                        echo "Login à Docker Hub..."
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        echo "Push de l'image..."
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }
        
        stage('Déployer avec Helm') {
            steps {
                script {
                    sh '''
                        helm upgrade --install mon-app $HELM_CHART_PATH \
                        --set image.repository=$DOCKER_IMAGE \
                        --set image.tag=latest
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Helm terminé'
        }
    }
}