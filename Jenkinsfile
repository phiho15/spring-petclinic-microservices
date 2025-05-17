def globalServiceChanged = []
def commitId = ''

pipeline {
    agent any
    environment {
        DOCKERHUB_USER = credentials('dockerhub-username') // Jenkins Credential ID
        DOCKERHUB_PASS = credentials('dockerhub-password') // Jenkins Credential ID
        DOCKERHUB_REPO = 'yourdockerhubuser/petclinic'     // ví dụ: huy123/petclinic
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

        stage('Build Docker Image') {
            when {
                expression { globalServiceChanged.size() > 0 }
            }
            steps {
                script {
                    withDockerRegistry([ credentialsId: 'dockerhub', url: '' ]) {
                        globalServiceChanged.each { svc ->
                            dir("${svc}") {
                                sh "docker build -t ${DOCKERHUB_REPO}-${svc}:${commitId} ."
                                sh "docker push ${DOCKERHUB_REPO}-${svc}:${commitId}"
                            }
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
