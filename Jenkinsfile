pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube'
        IMAGE_NAME = "vamshi589/boardgame"
        TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${TAG}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/vamshireddy903/Boardgame.git'
            }
        }

        stage('Git Pull') {
            steps {
                sh 'git pull origin main'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('File System Scan') {
            steps {
                sh 'trivy fs --format html -o trivy-fs-report.html . || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=boardgame-16 -Dsonar.projectName='BoardGame Jenkins Build 16' -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Publish Artifact to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: '677e6a02-5ce1-40f6-aec8-cee72cb5b9c7') {
                    sh 'mvn deploy'
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format html -o trivy-image-report.html ${IMAGE_NAME}:${TAG} || true"
            }
        }

        stage('Docker Image Push') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                }
            }
        }

        stage('Update Deployment YAML and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    script {
                        sh """
                        echo "Updating deployment-service.yaml with new image: ${FULL_IMAGE}"
                        sed -i "s|image:.*|image: ${FULL_IMAGE}|" deployment-service.yaml

                        git config --global user.email "devops@automation.com"
                        git config --global user.name "Jenkins Automation"

                        git add deployment-service.yaml
                        git diff-index --quiet HEAD || git commit -m "Update image to ${FULL_IMAGE}"

                        git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/vamshireddy903/Boardgame.git
                        git push origin main
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def status = currentBuild.result ?: 'SUCCESS'
                def color = status == 'SUCCESS' ? 'green' : 'red'

                def body = """
                <html>
                <body>
                    <div style="border: 4px solid ${color}; padding: 10px;">
                        <h2>Job: ${env.JOB_NAME} | Build #${env.BUILD_NUMBER}</h2>
                        <div style="background-color: ${color}; padding: 10px;">
                            <h3 style="color: white;">Pipeline Status: ${status}</h3>
                        </div>
                        <p>Check details: <a href="${env.BUILD_URL}">Console Output</a></p>
                    </div>
                </body>
                </html>
                """

                emailext(
                    subject: "Build ${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: body,
                    mimeType: 'text/html',
                    to: 'vamsir673@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    attachmentsPattern: 'trivy-*.html'
                )
            }
        }
    }
}
