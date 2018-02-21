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
         //If docker agent used here, target directory disapear after build...
         agent  {docker 'maven:3.5-alpine'}
         steps {
            sh 'mvn clean package'
            junit '**/target/surefire-reports/TEST-*.xml'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint:true
            stash name: "target", includes: "target/*"
            sh 'cp target/*.jar /opt/dump/'
         }
      }
      stage('Deploy') {
         steps {
               //input 'Do you approve the deployment?'
               echo 'deploying...'
               dir("target") {
                    unstash "target"
                }
               sh 'ls -la'
               sshagent (credentials: ['deploy_ssh']) {
                 sh "ssh -o StrictHostKeyChecking=no deploy@46.226.109.170 'echo $HOME'"
                 sh 'scp /opt/dump/*.jar deploy@46.226.109.170:/home/deploy/'
               }
         }
      }
   }
}
