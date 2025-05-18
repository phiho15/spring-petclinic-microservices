def globalServiceChanged = []
def commitId = ''

pipeline {
    agent any
    environment {
        DOCKERHUB_REPO = 'hykura/' // Thay bằng repo DockerHub của bạn
        KUBE_NAMESPACE = 'petclinic-dev'    // Namespace K8s bạn muốn deploy
    }

    parameters {
        string(name: 'VETS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch cho vets-service')
        string(name: 'CUSTOMERS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch cho customers-service')
        string(name: 'VISITS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch cho visits-service')
        string(name: 'API_GATEWAY_BRANCH', defaultValue: 'main', description: 'Branch cho api-gateway')
        string(name: 'CONFIG_SERVER_BRANCH', defaultValue: 'main', description: 'Branch cho config-server')
        string(name: 'DISCOVERY_SERVER_BRANCH', defaultValue: 'main', description: 'Branch cho discovery-server')
        string(name: 'ADMIN_SERVER_BRANCH', defaultValue: 'main', description: 'Branch cho admin-server')
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
                    def branches = [:]
        
                    globalServiceChanged.each { svc ->
                        branches[svc] = {
                            dir("${svc}") {
                                def imageTag = "${DOCKERHUB_REPO}${svc}:${commitId}"
                                echo "Building image: ${imageTag}"
                                sh '../mvnw clean install -P buildDocker -DskipTests'
                                sh "docker tag springcommunity/${svc}:latest ${imageTag}"
                                echo "Pushing image: ${imageTag}"
                                sh "docker push ${imageTag}"
                            }
                        }
                    }
        
                    // Run in parallel
                    parallel branches
                }
            }
        }
        
        // stage('Deploy to Kubernetes') {
        //     steps {
        //         script {
        //             // Map service to branch parameter
        //             def serviceBranchMap = [
        //                 "spring-petclinic-vets-service"      : params.VETS_SERVICE_BRANCH,
        //                 "spring-petclinic-customers-service" : params.CUSTOMERS_SERVICE_BRANCH,
        //                 "spring-petclinic-visits-service"    : params.VISITS_SERVICE_BRANCH,
        //                 "spring-petclinic-api-gateway"       : params.API_GATEWAY_BRANCH,
        //                 "spring-petclinic-config-server"     : params.CONFIG_SERVER_BRANCH,
        //                 "spring-petclinic-discovery-server"  : params.DISCOVERY_SERVER_BRANCH,
        //                 "spring-petclinic-admin-server"      : params.ADMIN_SERVER_BRANCH
        //             ]

        //             def serviceList = [
        //                 "spring-petclinic-admin-server",
        //                 "spring-petclinic-api-gateway",
        //                 "spring-petclinic-config-server",
        //                 "spring-petclinic-customers-service",
        //                 "spring-petclinic-discovery-server",
        //                 "spring-petclinic-vets-service",
        //                 "spring-petclinic-visits-service"
        //             ]

        //             serviceList.each { svc ->
        //                 def branch = serviceBranchMap[svc]
        //                 def tag = (branch == 'main') ? 'latest' : commitId
        //                 def image = "${DOCKERHUB_REPO}-${svc}:${tag}"

        //                 // Apply deployment with correct image tag
        //                 sh """
        //                 kubectl -n ${KUBE_NAMESPACE} set image deployment/${svc} ${svc}=${image} --record || \
        //                 kubectl -n ${KUBE_NAMESPACE} create deployment ${svc} --image=${image}
        //                 """
        //             }
        //         }
        //     }
        // }
    }

    post {
        success {
            echo "Build, push, and deploy completed for: ${globalServiceChanged}"
        }
    }
}
