pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ahmedhoucine0/mon-app-helm'
        HELM_VERSION = '3.12.0'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Installer Helm') {
            steps {
                sh '''
                    echo "Installation de Helm..."
                    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                    chmod 700 get_helm.sh
                    ./get_helm.sh --version v${HELM_VERSION}
                    helm version
                '''
            }
        }
        
        stage('Cloner le dépôt') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/ahmedhoucine/partie-helm.git'
            }
        }
        
        stage('Vérifier les fichiers Helm') {
            steps {
                sh '''
                    echo "=== Structure du dépôt ==="
                    find . -type f -name "*.yaml" -o -name "*.yml" -o -name "Chart.yaml" -o -name "values.yaml" | head -20
                    echo ""
                    echo "=== Recherche spécifique des charts Helm ==="
                    find . -name "Chart.yaml" -exec dirname {} \\; | head -10
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
                        echo "=== Déploiement avec Helm ==="
                        
                        # Trouver le chemin du chart Helm
                        CHART_PATH=$(find . -name "Chart.yaml" -exec dirname {} \\; | head -1)
                        
                        if [ -z "$CHART_PATH" ]; then
                            echo "ERREUR: Aucun chart Helm trouvé!"
                            echo "Création d'un chart Helm basique..."
                            
                            # Créer un chart Helm basique si aucun n'existe
                            helm create mon-app-chart
                            CHART_PATH="./mon-app-chart"
                            
                            # Modifier le values.yaml pour utiliser notre image
                            cat > ${CHART_PATH}/values.yaml << EOF
image:
  repository: ${DOCKER_IMAGE}
  tag: latest
  pullPolicy: IfNotPresent
EOF
                        fi
                        
                        echo "Chart Helm trouvé dans: $CHART_PATH"
                        
                        # Déployer avec Helm
                        helm upgrade --install mon-app $CHART_PATH \
                            --set image.repository=${DOCKER_IMAGE} \
                            --set image.tag=latest \
                            --atomic --timeout 5m
                        
                        echo "=== Vérification du déploiement ==="
                        helm list
                        kubectl get pods -l app=mon-app
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