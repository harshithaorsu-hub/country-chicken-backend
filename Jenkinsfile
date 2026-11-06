pipeline {
    agent any

    tools {
        maven 'maven3.9.16'
        jdk 'java21'
    }

    environment {
        APP_NAME         = 'country-chicken-backend'

        NEXUS_MAVEN_URL  = '98.81.230.204:8081'
        NEXUS_DOCKER_URL = '98.81.230.204:8081'

        MAVEN_REPO       = 'maven-releases'
        DOCKER_REPO      = 'docker-releases'

        GROUP_ID         = 'com.countrychicken'
        VERSION          = ''
        JAR_NAME         = ''
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/harshithaorsu-hub/country-chicken-backend.git'
            }
        }

        stage('Set Version') {
            steps {
                script {
                    VERSION = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    if (!VERSION) {
                        error "❌ Version not found from pom.xml"
                    }

                    JAR_NAME = "${APP_NAME}-${VERSION}.jar"
                    echo "✅ Version: ${VERSION}"
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload JAR to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl:"44.204.187.63:8081",
                    groupId: "com.countrychicken",
                    version: "VERSION",
                    repository: "Maven-Nexus-Repo",
                    credentialsId: 'nexus-credentials',
                    artifacts: [
                        [
                            artifactId: "country-chicken-backend",
                            classifier: '',
                            file: "target/${JAR_NAME}",
                            type: 'jar'
                        ]
                    ]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build \
                  -t ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${VERSION} \
                  -t ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:latest .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-nexus-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """

                    echo "$DOCKER_PASS" | docker login ${NEXUS_DOCKER_URL} -u "$DOCKER_USER" --password-stdin

                    docker push ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${VERSION}
                    docker push ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:latest

                    docker logout ${NEXUS_DOCKER_URL}
                    """

                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & Push Successful"
        }
        failure {
            echo "❌ Build Failed"
        }
        always {
            sh 'docker system prune -f'
            cleanWs()
        }
    }
}
