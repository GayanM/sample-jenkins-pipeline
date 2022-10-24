pipeline {
    agent any
    environment {
        USER_CREDENTIALS_NEXUS = credentials('GBoss-nexus')
        USER_CREDENTIALS_GIT = credentials('GBoss')

    }
    stages {
        stage('Clone Repo') {
            steps {
                echo "test"
                git branch: '${SOURCE_BRANCH}', changelog: false, credentialsId: 'GBoss', poll: false, url: 'https://github.com/Icurity/lab-gw-kong.git'
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
                sh 'docker login 65.21.125.148:9001/icurity/ -u $USER_CREDENTIALS_NEXUS_USR -p $USER_CREDENTIALS_NEXUS_PSW'
                sh 'docker tag ${APPLICATION}:${TAG} 65.21.125.148:9001/icurity/${APPLICATION}:${TAG}'
                sh 'docker push 65.21.125.148:9001/icurity/${APPLICATION}:${TAG}'    
                sh 'docker login 65.21.125.148:9001/icurity/ -u $USER_CREDENTIALS_NEXUS_USR -p $USER_CREDENTIALS_NEXUS_PSW'            }
        }  
        stage('Clone Helm') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'GBoss', poll: false, url: 'https://github.com/Icurity/lab_devops.git'
            }
        }
        stage('Replace docker tag') {
            steps {
                script{
                    def text = readYaml file:"${WORKSPACE}/iac/helms/kong/values.yaml"
                    text.kong.image.tag = "${TAG}"
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
