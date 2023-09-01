pipeline {
    agent any
    
    environment {
        ACR_NAME = "myegistry" 
        DOCKER_IMAGE_NAME = "my-web-app"
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        GITHUB_REPO = "https://github.com/yeddulaswetha/AKS.git"
        AKS_CLUSTER_NAME = "myakscluster" 
        KUBERNETES_CONFIG = credentials('kubeconfig') 
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: "${GITHUB_REPO}"]]])
                    sh "docker build -t ${ACR_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }
stage('Debugging') {
    steps {
        script {
            echo "Service Principal ID: a533fc9d-4729-4733-ba72-7a53f3411f4c"
            echo "Client Secret: IUQ8Q~mcskURjDXYMeGb9Qg6RRHB_F1qmUnrpbWf"
            echo "Tenant ID: e28e35a1-9c39-4f76-bca7-ede584f84f50"
        }
    }
}

     stage('Azure Login (Interactive)') {
    steps {
        script {
            sh "az login"
        }
    }
}

        stage('Azure Login') {
    steps {
        script {
            sh """
            az login --service-principal -u a533fc9d-4729-4733-ba72-7a53f3411f4c \
            -p IUQ8Q~mcskURjDXYMeGb9Qg6RRHB_F1qmUnrpbWf --tenant e28e35a1-9c39-4f76-bca7-ede584f84f50
            """
        }
    }
}

  
        stage('Push to ACR') {
            steps {
                script {
                    sh "az acr login --name ${ACR_NAME}"
                    sh "docker push ${ACR_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }
        
        stage('Deploy to AKS') {
            steps {
                script {
                    
                    writeFile(file: 'kubeconfig', text: KUBERNETES_CONFIG)
                    
                    sh "kubectl --kubeconfig=kubeconfig apply -f deployment.yaml"
                }
            }
        }
    }
}
