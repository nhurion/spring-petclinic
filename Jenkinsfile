
node {

   stage('Clone Repository') {
        // Get some code from a GitHub repository
        git 'https://github.com/nhurion/spring-petclinic.git'
    
   }
   stage('Build Maven Image') {
        docker.build("maven-build")
   }
   
   stage('Run Maven Container') {
       
        //Remove maven-build-container if it exisits
        sh " docker rm -f maven-build-container || true "
        
        //Run maven image
        sh "docker run --rm --name maven-build-container maven-build"
   }
   
   stage('Deploy Spring Boot Application') {
        
         //Remove maven-build-container if it exisits
        sh " docker rm -f java-deploy-container || true"
       
        sh "docker run --name java-deploy-container --volumes-from maven-build-container -d -p 8080:8090 nhurion/petclinic-deploy"
   }

}
