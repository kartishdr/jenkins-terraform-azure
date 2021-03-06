pipeline {
  agent {
      node {
          label 'master'
      }
  }

  environment {
      subscription_id = credentials('azure-subscription-id')
      tenant_id = credentials('azure-tenant-id')
      client_id = credentials('client-id-jenkins-sp')
      client_secret = credentials('client-secret-jenkins-sp')
      tfstate_rg = "tfstate"
      tfstate_sa = "tfstate90876"
      tfstate_sa_key = credentials('tfstate-storage-key')
      tfstate_container = "jenkinstf"
  }
  stages {
    stage('Terraform init') {
      steps {
            sh '''
               docker run -v /var/lib/jenkins/workspace/TerraformAzure:/var/lib/jenkins/workspace/TerraformAzure:rw,z \
                          -w /app \
                          -v /var/lib/jenkins/workspace/TerraformAzure:/app \
                          -e ARM_SUBSCRIPTION_ID="${subscription_id}" \
                          -e ARM_TENANT_ID="${tenant_id}" \
                          -e ARM_CLIENT_ID="${client_id}" \
                          -e ARM_CLIENT_SECRET="${client_secret}" \
                          hashicorp/terraform:light \
                          init -input=false -backend-config="resource_group_name=${tfstate_rg}" \
                                            -backend-config="storage_account_name=${tfstate_sa}" \
                                            -backend-config="container_name=${tfstate_container}" \
                                            -backend-config="key=${tfstate_sa_key}" \
                                            -backend-config="access_key=${tfstate_sa_key}"
            '''
      }
    }
    stage('Terraform plan') {
      steps {
            sh '''
               docker run -v /var/lib/jenkins/workspace/TerraformAzure:/var/lib/jenkins/workspace/TerraformAzure:rw,z \
                       -w /app \
                       -v /var/lib/jenkins/workspace/TerraformAzure:/app \
                       -e ARM_SUBSCRIPTION_ID="${subscription_id}" \
                       -e ARM_TENANT_ID="${tenant_id}" \
                       -e ARM_CLIENT_ID="${client_id}" \
                       -e ARM_CLIENT_SECRET="${client_secret}" \
                       hashicorp/terraform:light \
               plan -out=tfplan -input=false
            '''
            script {
                  timeout(time: 10, unit: 'MINUTES') {
                    input(id: "Deploy Gate", message: "Apply ${params.project_name}?", ok: 'Apply')
                  }
            }
      }
    }
    stage('Terraform apply') {
      steps {
            sh  '''
                docker run -v /var/lib/jenkins/workspace/TerraformAzure:/var/lib/jenkins/workspace/TerraformAzure:rw,z \
                       -w /app \
                       -v /var/lib/jenkins/workspace/TerraformAzure:/app \
                       -e ARM_SUBSCRIPTION_ID="${subscription_id}" \
                       -e ARM_TENANT_ID="${tenant_id}" \
                       -e ARM_CLIENT_ID="${client_id}" \
                       -e ARM_CLIENT_SECRET="${client_secret}" \
                       hashicorp/terraform:light \
                apply -lock=false -input=false tfplan
            '''
      }
    }
  }
}
