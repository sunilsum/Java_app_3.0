pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['create', 'delete'], description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'sunil729')
    }

    stages {

        stage('Git Checkout') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        gitCheckout(
                            branch: "main",
                            url: "https://github.com/sunilsum/Java_app_3.0.git"
                        )
                    }
                }
            }
        }

        stage('Unit Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        mvnTest()
                    }
                }
            }
        }

        stage('Integration Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        mvnIntegrationTest()
                    }
                }
            }
        }

        stage('Static code analysis: Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        def SonarQubecredentialsId = 'sonarqube-api'
                        statiCodeAnalysis(SonarQubecredentialsId)
                    }
                }
            }
        }

        stage('Quality Gate Status Check : Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        def SonarQubecredentialsId = 'sonarqube-api'
                        QualityGateStatus(SonarQubecredentialsId)
                    }
                }
            }
        }

        stage('Maven Build : maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        mvnBuild()
                    }
                }
            }
        }

        stage('Docker Image Build') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                    }
                }
            }
        }

        stage('Docker Image Scan: trivy') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        dockerImageScan("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                    }
                }
            }
        }

        stage('Docker Image Push : DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Log in to Docker Hub using credentials
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            docker.withRegistry('https://index.docker.io/v1/', "$DOCKER_USER:$DOCKER_PASS") {
                                // Push the Docker image to Docker Hub
                                dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Image Cleanup : DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        dockerImageCleanup("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                    }
                }
            }
        }
    }
}
