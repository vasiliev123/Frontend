def imageName = "vasilievpl123/frontend"
def dockerTag = ""
def dockerRegistry = ""
def registryCredentials = "dockerhub"

pipeline {
    agent {
        label 'agent'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = "1"
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Download repo') {
            steps {
                git branch: 'main', url: 'https://github.com/vasiliev123/Frontend'
            }
        }
        stage('Unit tests') {
            steps {
                sh '''pip3 install -r requirements.txt
                    python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'''
            }
        }
        stage('Build + SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Build docker image') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}"
                    appImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
        stage('Push docker image') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        appImage.push()
                        appImage.push("latest")
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success {
            build job: 'App_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
        }
    }
}
