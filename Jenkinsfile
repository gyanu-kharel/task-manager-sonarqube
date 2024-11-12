pipeline {
    agent any
    
    environment {
        // Set these to match your specific SonarQube and Nexus configurations
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-token') // You can store the SonarQube token in Jenkins credentials
        NEXUS_URL = 'http://nexus:8081/repository/maven-releases/'
        NEXUS_CREDENTIALS = credentials('nexus-credentials') // Store Nexus credentials in Jenkins credentials
        PROJECT_NAME = 'task-manager'
        VERSION = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from the Git repository
                git 'https://github.com/gyanu-kharel/task-manager-sonarqube'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install dependencies
                script {
                    sh 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                // Run the build (e.g., Webpack)
                script {
                    sh 'npm run build'  // Replace with your build command if different
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis on the codebase
                script {
                    sh """
                    sonar-scanner \
                    -Dsonar.projectKey=${PROJECT_NAME} \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Wait for the quality gate result from SonarQube
                script {
                    def qg = waitForQualityGate() // This will pause the pipeline and wait for SonarQube to process
                    if (qg.status != 'OK') {
                        error "SonarQube Quality Gate failed: ${qg.status}"
                    }
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                // Archive the built artifact
                script {
                    sh 'tar -czvf build-artifact.tar.gz dist/*'  // Adjust this to your actual build artifact
                    archiveArtifacts artifacts: 'build-artifact.tar.gz', allowEmptyArchive: true
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                // Deploy artifact to Nexus repository
                script {
                    sh """
                    curl -u ${NEXUS_CREDENTIALS_USR}:${NEXUS_CREDENTIALS_PSW} \
                    --upload-file build-artifact.tar.gz \
                    ${NEXUS_URL}${PROJECT_NAME}-${VERSION}.tar.gz
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment were successful!'
        }
        failure {
            echo 'There was an issue with the pipeline.'
        }
    }
}
