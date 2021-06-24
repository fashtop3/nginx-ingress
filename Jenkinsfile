pipeline {
    agent any

    parameters {  
        string(name: 'NAMESPACE', defaultValue: 'ingress-basic', description: 'namespace to deploy nginx') //assumes namespace exists
        string(name: 'REPLICAS', defaultValue: '2', description: 'number of nginx replicas')
        string(name: 'AKS_CLUSTER', defaultValue: 'my-cluster', description: 'kubernetes cluster name')
        string(name: 'RESOURCE_GRP', defaultValue: 'my-resource-group', description: 'resource group name')
    }


    triggers {
        //check for change logs in the repo
        pollSCM '''TZ=Africa/Lagos
                    * * * * *'''
    }

    stages {
        stage('Prepare Environment') {
            /*
            * Using azure as a case study
            * environment set by jenkins and not exposed to the logs - $AZURE_CLIENT_ID, $AZURE_CLIENT_SECRET, $AZURE_TENANT_ID
            * Login to azure CLI
            */
            steps{
                withCredentials([azureServicePrincipal('azure_sp_id')]) {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
                    sh 'az account set -s $AZURE_SUBSCRIPTION_ID'
                }
            }
        }

        stage('Add update Helm Repo') {
            steps{
                sh 'helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx'
                sh 'helm repo update'
            }
        }

        stage('Deploy') {
            when {
                branch 'main' //checks if the current build is on 'main bramch'
            }
            steps {
                /*
                * Get rbac kubernetes credentials to allow authenticating and applying changes. 
                */

                withCredentials([azureServicePrincipal('azure-aks-admin-sp')]) {
                    // specifiy resouurce group and cluster to authenticate with
                    sh "az aks get-credentials --resource-group $RESOURCE_GRP --name $AKS_CLUSTER --admin"
                }

                //deploys to specified namespace
                sh 'helm install nginx-ingress ingress-nginx/ingress-nginx --namespace $NAMESPACE --set controller.replicaCount=$REPLICAS'
            }
        }
    }
}