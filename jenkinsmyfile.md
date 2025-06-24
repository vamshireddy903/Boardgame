pipeline {
    agent any

    tools{
        java 'jdk17'
        maven 'maven3'
    }
    
    enviornment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/vamshireddy903/Boardgame.git'
            }
        }
        
         stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
         stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
         stage('File System scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
         stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BaordGame \
                          -Dsonar.java.binaries=. '''
            }
        }
        
        stage('Quality Gate') {
           steps {
           script {
            waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
    }
}
      stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish artifacts to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'Jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                         sh "mvn deploy"
}
            }
        }
        stage('Build Docker iamge and tag') {
            steps {
                sh "mvn compile"
            }
        }
        
         stage('Build Docker iamge and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh " docker build -d vamshi589/my-imgage:latest . "
                     }
                }
            }
        }
        stage('Docker image scan') {
            steps {
                sh "trivy --format table -o trivy-fs-report.html vamshi589/my-imgage:latest"
            }
        }
         stage('Docker image push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh " docker push vamshi589/my-imgage:latest"
                     }
                }
            }
        }
    }
}
