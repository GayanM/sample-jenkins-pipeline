def gv
pipeline {
    agent any
    environment {
        USER_CREDENTIALS = credentials('GBoss-nexus')
    }
    stages {
        stage('init'){
            steps {
                script {
                    gv = load "script.groovy"
                    gv.buildApp()
                }
            }
        }
        stage('Clone Repo') {
            steps {
                withGroovy{
                    sh 'groovy --version'
                }
                sh "rm -r *"
                git branch: 'main', changelog: false, credentialsId: 'GBoss', poll: false, url: 'https://github.com/Icurity/lab-gw-kong.git'
                sh "ls"
            }
        }
        stage('Building Image') {
            steps {
                script {
                    dockerImage = docker.build "${APPLICATION}:${TAG}"
                }
            }
        }
        stage('Pushing to Nexus') {
            steps {
                sh 'docker login 65.21.125.148:9001/icurity/ -u $USER_CREDENTIALS_USR -p $USER_CREDENTIALS_PSW'
                sh 'docker tag ${APPLICATION}:${TAG} 65.21.125.148:9001/icurity/${APPLICATION}:${TAG}'
                sh 'docker push 65.21.125.148:9001/icurity/${APPLICATION}:${TAG}'            }
        }    
    }
}
