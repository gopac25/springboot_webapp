pipeline {
   agent any

   environment {
     ORGANIZATION_NAME = "gopac25"
     YOUR_DOCKERHUB_USERNAME = "gopac"

     SERVICE_NAME = "springboot"
     registry="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:latest"
     registryCredential = 'dockerhub'
     dockerImage = ''
   }
   
   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
         script {
            sh "mvn clean package"
         }
         }
      }
      stage('Code Analysis') {
         steps {
         script {
           sh "mvn sonar:sonar"
         }
         }
      }
      
           stage('Deploy to Nexus') {
         steps {
         script {
          sh "mvn deploy"
         }
         }
      }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
      stage('Deploy to k8s'){
          steps{
             sshagent(['kops-machine']) {
                    sh 'envsubst < ${WORKSPACE}/deploy.yaml'
                    sh "scp -o StrictHostKeyChecking=no deploy.yaml ec2-user@13.233.215.97:/home/ec2-user/"
                   script{
                       try{
                            sh "ssh ec2-user@13.233.215.97 kubectl apply -f ."
                      }catch(error){
                            sh "ssh ec2-user@13.233.215.97 kubectl create -f ."
                        }
                    }
                }
            }
      }
   }
}