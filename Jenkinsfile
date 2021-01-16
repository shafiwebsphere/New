pipeline {
  
  parameters {
    choice choices: ['create', 'destroy'], description: 'Create or destroy the eks cluster', name: 'action'
    string defaultValue: 'unzer_cluster', description: 'EKS cluster name: unzer_cluster', name: 'cluster', trim: true
              }
 agent any
       tools {
         terraform 'terraform'
          }
 stages {
     stage('checkout') {
	steps {
            git credentialsId: 'shafiweb123', url: 'https://github.com/shafiweb123/unzer123.git'
             }
           }
     stage('Set Terraform path') {
 		steps{
		  script {
                def tfHome = tool name: 'terraform'
                env.PATH = "${tfHome}:${env.PATH}"
           }
		  sh 'terraform -version'
          }
          }
     stage('Terraform plan) {
          when {
        expression { params.action == 'create' }
		}	
            steps {
                   script{
		withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_Credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
				sh """
					terraform init
					terraform workspace new ${params.cluster} || true
					terraform workspace select ${params.cluster}
					terraform plan \
						-var cluster-name=${params.cluster} \
						-out ${plan}
					echo ${params.cluster}
				"""
				}
		          }
                  }
           }
     stage('Terraform Apply') {
		      when {
        expression { params.action == 'create' }
		}	
		steps {
			script {
				withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_Credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
				if (fileExists('$HOME/.kube')) {
					echo '.kube Directory Exists'
				} else {
				sh 'mkdir -p $HOME/.kube'
				}
				sh """
					terraform apply -input=false -auto-approve ${plan}
					terraform output kubeconfig > $HOME/.kube/config
				"""
				sh 'sudo + $(id -u):$(id -g) $HOME/.kube/config'
				sleep 30
				sh 'kubectl get nodes'
				
			}
        }
      }
           }
     stage('Terraform Destroy') {
      when {
        expression { params.action == 'destroy' }
      }
      steps {
        script {
			
				withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_Credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
				sh """
				terraform workspace select ${params.cluster}
				terraform destroy -auto-approve
				"""
				}
			
        }
      }
           }
       }
   }
