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
         }
      }
      stage('Deploy') {
         steps {
               //input 'Do you approve the deployment?'
               echo 'deploying...'
               //dir("target") {
                    unstash "target"
                //}
               sh 'ls -la'
               sh 'ls ./target -la'
               sshagent (credentials: ['deploy_ssh']) {
                 sh "ssh -o StrictHostKeyChecking=no deploy@46.226.109.170 'echo $HOME'"
                 sh 'scp target/*.jar deploy@46.226.109.170:/home/deploy/'
                 sh "ssh deploy@46.226.109.170 'nohup java -jar /home/deploy/spring-petclinic-1.5.1.jar -Dserver.port=8090 &'"
               }
         }
      }
   }
}
