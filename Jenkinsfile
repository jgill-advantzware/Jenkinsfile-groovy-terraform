#!/usr/bin/env groovy

library 'notify@v1.0.0'

pipeline {
  agent {
    label 'master'
  }
  options {
    ansiColor('xterm')
  }
  stages {
     
    stage('Plan') {

      when {
        not { branch 'master' }
      }
      steps {
        bitbucketStatusNotify(buildState: 'INPROGRESS')
        notifyTerraform('IN PROGRESS', 'Terraform plan starting')
        script {
          def terraformPath = tool name: 'Terraform-0.13.7', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
          sh "${terraformPath}/terraform --version"
          sh "${terraformPath}/terraform init -input=false"
          sh "${terraformPath}/terraform validate"
          sh "${terraformPath}/terraform plan -input=false"
        }
      }
      post {
        always {
          deleteDir()
        }
        failure {
          bitbucketStatusNotify(buildState: 'FAILED')
          notifyTerraform('FAILURE', 'Terraform plan failed')
        }
        success {
          bitbucketStatusNotify(buildState: 'SUCCESSFUL')
          notifyTerraform('SUCCESS', 'Terraform plan succeeded')
        }
      }
    }
    stage('Apply') {
      when { 
        branch 'master'
      }
      steps {
        bitbucketStatusNotify(buildState: 'INPROGRESS')
        timeout(time: 8, unit: 'HOURS') {
          script {
            def terraformPath = tool name: 'Terraform-0.13.7', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
            sh "${terraformPath}/terraform init -input=false"
            sh "${terraformPath}/terraform validate"
            sh "${terraformPath}/terraform plan -input=false -out terraform.tfplan"
            notifyTerraform('APPLY PENDING', 'Terraform plan succeeded, waiting for approval')
            input 'Execute terraform apply?'
            notifyTerraform('APPLY APPROVED', "Terraform apply approved")
            sh "${terraformPath}/terraform apply -input=false -auto-approve terraform.tfplan"
          }
        }
      }
      post {
        always {
          deleteDir()
        }
        failure {
          bitbucketStatusNotify(buildState: 'FAILED')
          notifyTerraform('FAILURE', 'Terraform plan or apply failed')
        }
        success {
          bitbucketStatusNotify(buildState: 'SUCCESSFUL')
          notifyTerraform('SUCCESS', 'Terraform apply succeeded')
        }
      }
    }
  }
}
