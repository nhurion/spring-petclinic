#!groovy



pipeline {
   agent any

    environment {
        projectName = 'pet'
        deploymentServer = '46.226.109.170'
    }

   stages {
      stage('Clone Repository') {
         steps {
           // Get some code from a GitHub repository
           git 'https://github.com/nhurion/spring-petclinic.git'
         }
      }
      stage('Build') {
         //If docker agent used here, target directory disapear after build...
         agent  {docker 'maven:3.5-alpine'}
         steps {
            sh 'mvn clean package -T2'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint:true
            stash name: "target", includes: "target/*"
         }
      }
       stage('Deploy Dev') {
           steps {
               script {
                   filePath = "/opt/projects/dev/${projectName}/"
               }
               //input 'Do you approve the deployment?'
               echo 'deploying...'
               unstash "target"
               sshagent (credentials: ['deploy_ssh']) {
                   sh "ssh -o StrictHostKeyChecking=no deploy@${deploymentServer} 'echo hello'"
                   sh "ssh -f deploy@${deploymentServer} 'pkill -e -f dev/${projectName} || true' "
                   sh "scp target/*.jar deploy@${deploymentServer}:${filePath}"
                   sh "ssh -f deploy@${deploymentServer} 'cd ${filePath} && nohup java -jar ${filePath}spring-petclinic-1.5.1.jar & '"
               }
           }
       }
       stage('Smoke test dev') {
           steps {
               script{
                   workspacePath = pwd()
               }
               sh 'sleep 60'
               sh "curl --retry-delay 10 --retry 5 http://${projectName}.dev.hurion.be/manage/info -o ${workspacePath}/info-dev.json"
               archiveArtifacts artifacts: "info-dev.json", fingerprint:true
           }
       }
       stage('Deploy Test') {
           steps {
               script {
                   filePath = "/opt/projects/test/${projectName}/"
               }
               //input 'Do you approve the deployment to test?'
               echo 'deploying...'
               unstash "target"
               sshagent (credentials: ['deploy_ssh']) {
                   sh "ssh -o StrictHostKeyChecking=no deploy@${deploymentServer} 'echo hello'"
                   sh "ssh -f deploy@${deploymentServer} 'pkill -e -f test/${projectName} || true' "
                   sh "scp target/*.jar deploy@${deploymentServer}:${filePath}"
                   sh "ssh -f deploy@${deploymentServer} 'cd ${filePath} && nohup java -jar ${filePath}spring-petclinic-1.5.1.jar & '"
               }
           }
       }
       stage('Smoke test test') {
           steps {
               script{
                   workspacePath = pwd()
               }
               sh 'sleep 60'
               sh "curl --retry-delay 10 --retry 5 http://${projectName}.test.hurion.be/manage/info -o ${workspacePath}/info-test.json"
               archiveArtifacts artifacts: "info-test.json", fingerprint:true
           }
       }
   }
}
