pipeline {
    agent any
     environment {
        max = 500000
        random_numi = "${Math.abs(new Random().nextInt(max+15))}"
        def random_num = String.valueOf(random_numi)
        def RESOURCE_GROUP = "res-group-vito"
        def CLUSTER_NAME = "cluster-vito"
        def USER_NP = "userpool1"
        def LOCATION = "italynorth"
    }
    stages {
            stage('Azure Login & Action') {
            steps {
                // Vincula las credenciales almacenadas de forma segura
                withCredentials([azureServicePrincipal('azure-service-principal-creden')]) {
                    sh '''
                        # Login en Azure utilizando el Service Principal
                        az login --service-principal \
                          -u "$AZURE_CLIENT_ID" \
                          -p "$AZURE_CLIENT_SECRET" \
                          -t "$AZURE_TENANT_ID"
                        
                        # Establecer la suscripción por defecto
                        az account set -s "$AZURE_SUBSCRIPTION_ID"
                        
                        # Ejemplo de comando para validar que estás conectado
                        az group list -o table
                    '''
                }
            }
        }
        stage('Crea Resource Group') {
            steps {
                    sh 'az group create --name $RESOURCE_GROUP-$random_num --location $LOCATION'
                   }
            }
        stage('Crea AKS') {
            steps {
                    sh 'az aks create --resource-group $RESOURCE_GROUP-$random_num --name $CLUSTER_NAME-$random_num --node-count 1 --generate-ssh-keys'
            }
        }
            stage('CreaNodos'){
                steps {
                    sh 'az aks nodepool add --resource-group $RESOURCE_GROUP-$random_num --cluster-name $CLUSTER_NAME-$random_num --name $USER_NP --node-count 1 --mode User'
            }

            }
            stage('SetCred') {
                steps {
                sh 'az aks get-credentials --resource-group $RESOURCE_GROUP-$random_num --name $CLUSTER_NAME-$random_num'
                }
            }
            stage('CreaApp') {
                steps {
                    sh 'kubectl apply -f aks-store-quickstart.yaml'
                 }
            }
    }
}
