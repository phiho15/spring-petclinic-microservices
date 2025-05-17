def globalServiceChanged = []
def commitId = ''

pipeline {
    agent any
    environment {
        DOCKERHUB_REPO = 'ntquan87/petclinic' // ví dụ repo DockerHub của bạn
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                    commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def changes = sh(script: 'git diff --name-only HEAD~1 HEAD', returnStdout: true).trim()

                    def serviceList = [
                        "spring-petclinic-admin-server",
                        "spring-petclinic-api-gateway",
                        "spring-petclinic-config-server",
                        "spring-petclinic-customers-service",
                        "spring-petclinic-discovery-server",
                        "spring-petclinic-genai-service",
                        "spring-petclinic-vets-service",
                        "spring-petclinic-visits-service"
                    ]

                    for (svc in serviceList) {
                        if (changes.contains("${svc}/")) {
                            globalServiceChanged << svc
                        }
                    }

                    echo "Changed services: ${globalServiceChanged}"
                    echo "Commit ID: ${commitId}"
                }
            }
        }

        stage('Build & Push Docker Images') {
            when {
                expression { globalServiceChanged.size() > 0 }
            }
            steps {
                script {
                    globalServiceChanged.each { svc ->
                        dir("${svc}") {
                            def imageTag = "${DOCKERHUB_REPO}-${svc}:${commitId}"
                            echo "Building image: ${imageTag}"
                            sh "docker build -t ${imageTag} ."
                            echo "Pushing image: ${imageTag}"
                            sh "docker push ${imageTag}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build and push completed for: ${globalServiceChanged}"
        }
    }
}
