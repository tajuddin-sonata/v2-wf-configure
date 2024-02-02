pipeline {
    agent any

    parameters{
        choice(name: 'ENVIRONMENT', choices:[
            'dev',
            'staging',
            'prod'],
            description: 'Choose which environment to deploy to.')
        string(name: 'AZURE_FUNCTION_APP_NAME', defaultValue: 'jenkins-wf-configure', description: 'The name of FunctionApp to deploy')
        string(name: 'RESOURCE_GROUP_NAME', defaultValue: 'CCA-DEV', description: 'Azure Resource Group in which the FunctionApp need to deploy')
        string(name: 'FUNC_STORAGE_ACCOUNT_NAME', defaultValue: 'ccadevfunctionappstgacc', description: 'select the existing Storage account name for Func App or create new')
        string(name: 'REGION', defaultValue: 'CentralIndia', description: 'Region to Deploy to.')
        choice(name: 'SKU', choices:[
            'B1', 'B2', 'B3', 
            'S1','S2', 'S3', 
            'P1V3','P2V3', 'P3V3'], 
            description: 'ASP SKU.')
        choice(name: 'PYTHON_RUNTIME_VERSION', choices:[
            '3.9',
            '3.10',
            '3.11'],
            description: 'Python runtime version.')
    }

    environment {
        AZURE_CLIENT_ID = credentials('azurerm_client_id')
        AZURE_CLIENT_SECRET = credentials('azurerm_client_secret')
        AZURE_TENANT_ID = credentials('azurerm_tenant_id')
        ZIP_FILE_NAME = "function_code.zip"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                // Install Pip
                sh 'sudo yum install -y python3-pip'

                // Install project dependencies
                sh 'pip3 install -r requirements.txt -t .'
            }
        }

        stage('Package Code') {
            steps {
                sh "zip -r ${ZIP_FILE_NAME} ."
            }
        }

        stage('Create FunctionApp') {
            steps {
                // Create ASP for functionApp
                sh "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                sh "az appservice plan create --name ${params.AZURE_FUNCTION_APP_NAME}-plan --resource-group ${params.RESOURCE_GROUP_NAME} --sku ${params.SKU} --is-linux --location ${params.REGION}"
                
                // Create FunctionApp
                sh "az functionapp create --name ${params.AZURE_FUNCTION_APP_NAME} --resource-group ${params.RESOURCE_GROUP_NAME} --plan ${params.AZURE_FUNCTION_APP_NAME}-plan --runtime python --runtime-version ${params.PYTHON_RUNTIME_VERSION} --functions-version 4 --storage-account ${params.FUNC_STORAGE_ACCOUNT_NAME}"
            }
        }

        stage('Deploy to Azure Function App') {
            steps {
                script {
                    // Azure CLI commands to deploy the function code
                    sh "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    sh "az functionapp deployment source config-zip --src ${ZIP_FILE_NAME} --name ${params.AZURE_FUNCTION_APP_NAME} --resource-group ${params.RESOURCE_GROUP_NAME}"
                }
            }
        }
    }
}