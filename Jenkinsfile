pipeline {
    agent any

    environment {
        // Define SonarQube environment variables
        SONARQUBE_SERVER = 'sonar-server'  
        GITHUB_REPO = 'https://github.com/yahialm/CICD-pipeline-with-Jenkins-ArgoCD-Sonar-and-K8s.git' 
        SONAR_PROJECT_KEY = 'sqp_2417fd786eb483b86114e93c1afb3b8adf7e6310' 
        SONARQUBE_TOKEN = credentials('sonar-token')
        DOCKERHUB_CREDENTIALS = "docker-hub-credentials-id" 
        DOCKER_IMAGE_NAME = 'yahialm/spring'
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout the code from GitHub
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Build') {
            steps {
                // Give the permissions to mvnw
                sh 'chmod +x mvnw'

                // Build the Spring Boot project using Maven
                sh './mvnw clean install'
            }
        }

        stage('Test') {
            steps {
                // Run the tests using Maven
                sh './mvnw test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh "./mvnw sonar:sonar " +
                           "-Dsonar.projectKey=${SONAR_PROJECT_KEY}" +
                           "-Dsonar.login=${SONARQUBE_TOKEN}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry("https://registry.hub.docker.com", "${DOCKERHUB_CREDENTIALS}") {
                        // Push the Docker image to DockerHub
                        sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        // stage('Quality Gate') {
        //     // Wait for SonarQube's quality gate result before proceeding
        //     steps {
        //         script {
        //             timeout(time: 5, unit: 'MINUTES') {
        //                 waitForQualityGate abortPipeline: true
        //             }
        //         }
        //     }
        // }

    }

    post {
        always {
            // Archive the built artifacts and test results
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            junit '**/target/surefire-reports/*.xml'
        }
        failure {
            // Notify on failure
            echo 'Build failed!'
        }
        success {
            // Notify on success
            echo 'Build succeeded!'
        }
    }
}
