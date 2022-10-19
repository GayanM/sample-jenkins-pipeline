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
        }stage('Replace docker tag') {      
            steps {
                script{
                    def text = readYaml file:"${WORKSPACE}/iac/helms/kong/values.yaml"
                    text.image.tag = "${TAG}"
                    sh "rm -rf iac/helms/kong/values.yaml"
                    writeYaml file: 'iac/helms/kong/values.yaml', data: text
                    sh "cat iac/helms/kong/values.yaml"
                    sh 'git remote set-url origin https://$USER_CREDENTIALS_GIT_USR:$USER_CREDENTIALS_GIT_PSW@github.com/Icurity/lab_devops.git'
                    sh 'git config --global user.email "gmgunawardana@gmail.com"'
                    sh 'git config --global user.name "GayanM"'
                    sh "git add ."
                    sh 'git commit -am "Change values.yaml via Jenkins Pipeline"'
                    sh "git push origin main"
                }
            }
        }    
    }
}
