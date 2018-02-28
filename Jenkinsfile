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
                git branch: branch, url: scmUrl
            }
        }
        stage('Build') {
            //If docker agent used here, target directory disappear after build, so stash what you need to keep.
            agent { docker 'maven:3.5-alpine' }
            steps {
                sh 'mvn clean package -T2'
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                stash name: "target", includes: "target/*"
            }
        }
        parallel {
            stage('Deploy Dev') {
                steps {
                    deploy("${projectName}", "spring-petclinic-1.5.1.jar", "dev", false)
                    smokeTest("${projectName}", "dev")
                }
            }
            stage('Deploy Staging') {
                steps {
                    deploy("${projectName}", "spring-petclinic-1.5.1.jar", "test")
                    smokeTest("${projectName}", "test")
                }
            }
        }
        stage('Release') {
            steps {
                script {
                    env.RELEASE_SCOPE = input message: 'User input required', ok: 'Release!',
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'patch\nminor\nmajor', description: 'What is the release scope?')]
                }
                echo "releasing ${env.RELEASE_SCOPE}"
            }
        }
        stage('Deploy Prod') {
            steps {
                deploy("${projectName}", "spring-petclinic-1.5.1.jar", "prod")
                smokeTest("${projectName}", "prod")
            }
        }
    }
}
