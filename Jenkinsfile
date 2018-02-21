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
         environment { 
                DEPLOY = credentials('deploy') 
            }
         steps {
               input 'Do you approve the deployment?'
               echo 'deploying...'
               sh 'scp target/*.jar $DEPLOY@ci.hurion.be:/home/$DEPLOY_USR/pet/'
         }
      }
   }
}
