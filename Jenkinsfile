pipeline {
    agent any
    tools {
        terraform 'terraform'   
        ansible 'ansible'     
    }
    stages {
        stage('Checkout Code') {
            steps {
               checkout scm
            }
        }
        stage('Terraform Init & Plan') {
            when {
                anyOf {
                    branch 'main'
                    changeRequest()
                }
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding', 
                    credentialsId: 'aws-cred', 
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',  
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' 
                ]]) {
                dir('terraform'){
                        sh 'terraform init'
                        sh 'terraform force-unlock -force f728a7d5-ffdc-8b74-dae4-fe49c3ed1b47'
                        sh 'terraform plan'
                    }
                }
            }
        }
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding', 
                    credentialsId: 'aws-cred', 
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',  
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' 
                ]]) {
                dir('terraform'){
                        sh 'terraform apply -auto-approve'
                        echo '--- Checking Inventory File ---'
                        sh 'cat ../ansible/nexus/inventory.ini'
                    }
                }
            }
        }
        stage('Ansible Install Nexus') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding', 
                    credentialsId: 'aws-cred', 
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',  
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' 
                ]]) {
                dir('ansible/nexus'){
                        sh 'ansible-galaxy collection install amazon.aws || true'
                        echo "--- Running Ansible Playbook ---"
                        sh 'ansible-playbook -i inventory.ini playbook.yml'
                    }
                }
            }
        }
    }
}