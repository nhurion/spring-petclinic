#!groovy

@Library('nhu') _

pipeline {
   agent any

    environment {
        projectName = 'pet'
        deploymentServer = '46.226.109.170'
        branch = 'master'
        scmUrl = 'https://github.com/nhurion/spring-petclinic.git'
    }
   stages {
      stage('Clone Repository') {
         steps {
           // Get some code from a Git repository
           git  branch: branch, url: scmUrl
         }
      }
      stage('Build') {
         //If docker agent used here, target directory disappear after build, so stash what you need to keep.
         agent  {docker 'maven:3.5-alpine'}
         steps {
            sh 'mvn clean package -T2'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint:true
            stash name: "target", includes: "target/*"
         }
      }
       stage('Deploy Dev') {
           environment {
               deploymentEnvironment = 'dev'
           }
           steps {
               deploy (${projectName}, "spring-petclinic-1.5.1.jar", dev)
           }
       }
       stage('Smoke test dev') {
           environment {
               deploymentEnvironment = 'dev'
           }
           steps {
               script{
                   workspacePath = pwd()
               }
               sh 'sleep 60'
               sh "curl --retry-delay 10 --retry 5 http://${projectName}.${deploymentEnvironment}.hurion.be/manage/info -o ${workspacePath}/info-${deploymentEnvironment}.json"
               archiveArtifacts artifacts: "info-${deploymentEnvironment}.json", fingerprint:true
           }
       }
       stage('Deploy Staging') {
           environment {
               deploymentEnvironment = 'test'
           }
           steps {
               script {
                   filePath = "/opt/projects/test/${projectName}/"
               }
               //input "Do you approve the deployment to ${deploymentEnvironment}?"
               echo 'deploying...'
               unstash "target"
               sshagent (credentials: ['deploy_ssh']) {
                   sh "ssh -o StrictHostKeyChecking=no deploy@${deploymentServer} 'echo hello'"
                   sh "ssh -f deploy@${deploymentServer} 'pkill -e -f ${deploymentEnvironment}/${projectName} || true' "
                   sh "scp target/*.jar deploy@${deploymentServer}:${filePath}"
                   sh "ssh -f deploy@${deploymentServer} 'cd ${filePath} && nohup java -jar ${filePath}spring-petclinic-1.5.1.jar & '"
               }
           }
       }
       stage('Smoke test Staging') {
           environment {
               deploymentEnvironment = 'test'
           }
           steps {
               script{
                   workspacePath = pwd()
               }
               sh 'sleep 60'
               sh "curl --retry-delay 10 --retry 5 http://${projectName}.${deploymentEnvironment}.hurion.be/manage/info -o ${workspacePath}/info-${deploymentEnvironment}.json"
               archiveArtifacts artifacts: "info-${deploymentEnvironment}.json", fingerprint:true
           }
       }
   }
}
