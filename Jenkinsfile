pipeline {
   agent any
   stages {
      stage('Clone Repository') {
         steps {
           // Get some code from a GitHub repository
           git 'https://github.com/nhurion/spring-petclinic.git'
         }
      }
      stage('Build') {
         agent {docker 'maven:3.5-alpine'}
         steps {
            sh 'mvn clean package'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint:true
         }
      }
      stage('Deploy') {
         withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '<CREDENTIAL_ID>',
                           usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            steps {
                input 'Do you approve the deployment?'
                echo 'deploying...'
                sh 'scp target/*.jar $USERNAME:$PASSWORD@ci.hurion.be:/home/$USERNAME/pet/'
             }
         }
      }
   }
}
