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
        stage('Deploy Dev') {
            steps {
                deploy("${projectName}", "spring-petclinic-1.5.1.jar", "dev", false)
            }
        }
        stage('Smoke test dev') {
            steps {
                smokeTest("${projectName}", "dev")
            }
        }
        stage('Deploy Staging') {
            steps {
                deploy("${projectName}", "spring-petclinic-1.5.1.jar", "test")
            }
        }
        stage('Smoke test Staging') {
            steps {
                smokeTest("${projectName}", "test")
            }
        }
        stage('Ready for prod') {
            steps {
                script {
                    env.RELEASE_SCOPE = input message: 'User input required', ok: 'Release!',
                            parameters: [choice(name: 'RELEASE_SCOPE', choices: 'patch\nminor\nmajor', description: 'What is the release scope?')]
                }
                echo "${env.RELEASE_SCOPE}"
            }
        }
        stage('Release') {
            steps {
                echo "releasing ${env.RELEASE_SCOPE}"
            }
        }
        stage('Deploy Prod') {
            steps {
                deploy("${projectName}", "spring-petclinic-1.5.1.jar", "prod")
            }
        }
        stage('Smoke test Prod') {
            steps {
                smokeTest("${projectName}", "prod")
            }
        }
    }
}
