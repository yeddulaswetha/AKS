@Library('my-shared-library') _

pipeline {
    agent any

    environment {
        def envVars = mySharedLibrary.environment.setEnvironmentVariables()
        ACR_NAME = envVars.ACR_NAME
        DOCKER_IMAGE_NAME = envVars.DOCKER_IMAGE_NAME
        DOCKER_IMAGE_TAG = envVars.DOCKER_IMAGE_TAG
        GITHUB_REPO = envVars.GITHUB_REPO
        AKS_CLUSTER_NAME = envVars.AKS_CLUSTER_NAME
        KUBECONFIG_PATH = envVars.KUBECONFIG_PATH
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
                    mySharedLibrary.debugging.debug()
                }
            }
        }

        stage('Azure Login (Interactive)') {
            steps {
                script {
                    mySharedLibrary.azureLoginInteractive()
                }
            }
        }

        stage('Azure Login') {
            steps {
                script {
                    def servicePrincipalId = "77d92cb1-580e-4a67-96b9-44954361a2fd"
                    def clientSecret = "IUQ8Q~mcskURjDXYMeGb9Qg6RRHB_F1qmUnrpbWf"
                    def tenantId = "e28e35a1-9c39-4f76-bca7-ede584f84f50"
                    mySharedLibrary.azureLogin(servicePrincipalId, clientSecret, tenantId)
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
                    sh "kubectl --kubeconfig=${KUBECONFIG_PATH} apply -f deployment.yaml"
                }
            }
        }
    }
}
