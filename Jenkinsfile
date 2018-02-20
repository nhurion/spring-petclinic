
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
            
            def pom = readMavenPom file:'pom.xml'
            print pom.version
            env.version = pom.version
            
            junit '**/target/surefire-reports/TEST-*.xml'
         }
      }
      stage('Image') {
            steps {
                def app = docker.build "localhost:5000/petclinic-deploy:${env.version}"
                app.push()
            }
        }
   }
}
