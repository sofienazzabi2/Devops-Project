pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'AzureServiceCred' 
        CLUSTER_NAME = 'pfa-cluster'
        RESOURCE_NAME = 'kubernetes'
        DEP = 'deployment3.yaml'
    }
    
    stages {
        stage('Azure Authentication') {
            steps {
                script {
                    // Authenticate with Azure using service principal credentials
                    withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                        bat "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
                        bat "az account set --subscription ${AZURE_SUBSCRIPTION_ID}"
                        bat "az account show"
                    }
                }
            }
        }
        
        stage('Terraform init') {
            steps {
                script { 
                 bat 'terraform init'     
                }
            }
        }
         
        stage('Terraform apply') {
            steps {
                script {
                bat 'echo "Creating the infrastructure"'
                  bat 'terraform apply -auto-approve'
                }
            }
        } 
      

        stage('Configure kubectl') {
            steps {
                script {  
                    bat "az aks get-credentials --resource-group ${RESOURCE_NAME} --name ${CLUSTER_NAME}  --overwrite-existing"
                    bat 'kubectl config view'
                    bat 'kubectl get services'
                    bat 'kubectl get pods'
                    bat "helm version"
                    bat ' helm repo add prometheus-community https://prometheus-community.github.io/helm-charts'
                    bat '  helm repo add stable https://charts.helm.sh/stable'
                    bat 'helm repo update'
                    def output = bat(returnStdout: true, script: ' helm list -n default')
                    if (output.contains("prometheus")) {
                     bat 'echo "prometheus found!"'
                    } else {
                           bat 'echo "prometheus not found!"'
                           bat 'helm install prometheus  prometheus-community/kube-prometheus-stack   --replace'
                    }
                }
            }
        }
        stage('Add ArgoCD') {
            steps {
                script {
                   bat ' echo "Applying Argo"'
                   bat ' kubectl create namespace argocd'
                   bat ' kubectl apply -n argocd -f "https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml"'
                   bat ' kubectl get secrets -n argocd'
                   bat ' kubectl edit secret argocd-initial-admin-secret -n argocd'
                   bat ' kubectl patch svc argocd-server -n argocd --type=json -p="[{"op": "replace", "path": "/spec/type", "value": "LoadBalancer"}]"'

                }
            }
        }
        stage('Run Kubernetes Manifest') {
            steps {
                script {
                 //Run kubectl command to apply the Kubernetes manifest file
                 bat 'echo "Applying the deployment"'
                 bat "kubectl apply -f ${DEP}"
                 bat 'kubectl get services'
                }
            }
        }
        
        stage('Terraform destroy') {
            steps {
                script {  

                 bat 'echo "Destroying the infrastructure"'
//               bat 'terraform destroy --auto-approve'
                }
            }
        }

    }
}  
