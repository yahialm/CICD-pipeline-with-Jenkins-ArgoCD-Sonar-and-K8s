pipeline {
    agent any

    environment {
        // Define SonarQube environment variables
        SONARQUBE_SERVER = 'sonar-server'  
        GITHUB_REPO = 'https://github.com/yahialm/CICD-pipeline-with-Jenkins-ArgoCD-Sonar-and-K8s.git' 
        SONAR_PROJECT_KEY = 'sqp_2417fd786eb483b86114e93c1afb3b8adf7e6310' 
        SONARQUBE_TOKEN = credentials('sonar-token')
        NVD_API_KEY = credentials('NVD-API')
        DOCKERHUB_CREDENTIALS = "docker-hub-credentials-id" // This should be the actual ID used in docker.withRegistry, see: https://www.jenkins.io/doc/book/pipeline/docker/ last section on how to use withRegistry
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
                           "-Dsonar.projectKey=${SONAR_PROJECT_KEY} " +
                           "-Dsonar.login=${SONARQUBE_TOKEN}"
                    }
                }
            }
        }

        stage('OWASP Dependency Check Scan and Publish') {
            steps {
                script {
                    dependencyCheck additionalArguments: """
                    -o './' 
                    -s './'
                    -f 'ALL' 
                    --prettyPrint
                    --nvdApiKey ${NVD_API_KEY} """, odcInstallation: "dependency-check"

                    dependencyCheckPublisher pattern: 'dependency-check-report.html'
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

        stage('Trivy Scan') {
            steps {
                script {
                    // Define file path where the report should be saved
                    // def trivyReportFile = "trivy-report-${env.BUILD_NUMBER}.txt"
                    // Run Trivy to scan the Docker image
                    // def trivyOutput = sh(script: "trivy image ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} > ${trivyReportFile}", returnStdout: true).trim()
                    // Archive the report in Jenkins for later reference
                    // archiveArtifacts artifacts: trivyReportFile
                    // Display Trivy scan results
                    // println trivyOutput


                    // Save Trivy scan result as an HTML report
                    // Please use the path indicated @/usr/... as the default path to html.tpl in every project
                    // Trivy automatically install the templates there 
                    // DO NOT CHANGE THE PATH !!! Refer to: https://stackoverflow.com/a/76288013
                    def trivyHtmlReportFile = "trivy-report-${env.BUILD_NUMBER}.html"
                    sh """trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} > ${trivyHtmlReportFile}"""
                    
                    // Publish the HTML report (requires HTML Publisher Plugin)
                    publishHTML([
                        reportName: 'Trivy Security Scan',
                        reportDir: '',
                        reportFiles: trivyHtmlReportFile,
                        keepAll: true,
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        includes: '**/*'
                    ])
                    }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry([ credentialsId: "docker-hub-credentials-id", url: "https://index.docker.io/v1/" ]) {
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
             // dependencyCheckPublisher pattern: 'dependency-check-report/*.html'
             // junit '**/target/surefire-reports/*.xml'
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
