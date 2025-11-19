pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ahmedhoucine0/mon-app'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ahmedhoucine/tp3-devops.git'
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
        
        stage('Déployer sur Kubernetes') {
    steps {
        sh '''
            echo "=== Installation de kubectl ==="
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            mv kubectl /usr/local/bin/
            
            echo "=== Configuration pour accéder à Minikube sur l'hôte ==="
            # Obtenir l'adresse IP de l'hôte Docker
            HOST_IP=$(ip route | grep default | awk '{print $3}' || echo "host.docker.internal")
            echo "Adresse de l'hôte: $HOST_IP"
            
            # Créer un kubeconfig personnalisé pointant vers l'hôte
            cat > /tmp/kubeconfig-jenkins << EOF
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /tmp/minikube-ca.crt
    server: https://$HOST_IP:63980
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /tmp/minikube-client.crt
    client-key: /tmp/minikube-client.key
EOF
            
            export KUBECONFIG=/tmp/kubeconfig-jenkins
            
            echo "=== Vérification ==="
            kubectl cluster-info
            kubectl get nodes
        '''
    }
}
    }
    post {
        always {
            echo 'Pipeline terminé'
        }
        success {
            echo 'Déploiement réussi!'
        }
        failure {
            echo 'Échec du déploiement'
        }
    }
}



