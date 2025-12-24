pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
               checkout scm
            }
        }
        stage('Terraform Init & Plan') {

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
                        sh 'terraform plan'
                    }
                }
            }
        }
        stage('Terraform Apply') {

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
