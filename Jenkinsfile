
pipeline {
   agent {docker 'maven:3.5-alpine'}
   stages {
      stage('Clone Repository') {
         steps {
           // Get some code from a GitHub repository
           git 'https://github.com/nhurion/spring-petclinic.git'
         }
      }
      stage('Build') {
         steps {
            sh 'mvn clean package'
            junit '**/target/surfire-reports/TEST-*.xml'
         }
      }
   }
}
