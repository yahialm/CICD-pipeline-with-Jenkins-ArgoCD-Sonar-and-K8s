pipeline {
    agent any

    environment {
        // Define SonarQube environment variables
        SONARQUBE_SERVER = 'sonar-server'  // This is the name you gave to your SonarQube instance in Jenkins settings
        GITHUB_REPO = 'https://github.com/yahialm/CICD-pipeline-with-Jenkins-ArgoCD-Sonar-and-K8s.git'  // Replace with your repository URL
        SONAR_PROJECT_KEY = 'sqp_2417fd786eb483b86114e93c1afb3b8adf7e6310'  // Replace with your SonarQube project key
        SONARQUBE_TOKEN = credentials('sonar-token')  // Define SonarQube token stored in Jenkins credentials
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
                        sh './mvnw sonar:sonar ' +
                           '-Dsonar.projectKey=${SONAR_PROJECT_KEY} ' +
                           '-Dsonar.login=${SONARQUBE_TOKEN}'
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
