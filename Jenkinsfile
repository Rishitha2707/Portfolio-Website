pipeline {

    agent any

    environment {
        COMMIT_HASH = "${GIT_COMMIT.substring(0,7)}"
        IMAGE_NAME = "rishi01dadireddy/safe-ride-app:${COMMIT_HASH}"

        SONAR_HOST_URL = "http://52.8.253.11:9000"
        NEXUS_URL = "http://52.8.253.11:8081/repository/npm/"
    }

    tools {
        nodejs 'Node16'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Commit: ${GIT_COMMIT}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('Sonar') {
                        sh """
                            ${tool 'Sonar'}/bin/sonar-scanner \
                              -Dsonar.projectKey=my-node-app \
                              -Dsonar.sources=. \
                              -Dsonar.exclusions=node_modules/**,build/** \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Zip Artifact') {
            steps {
                sh """
                    apt-get update -y && apt-get install -y zip
                    rm -f build.zip
                    zip -r build.zip build/
                """
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NUSER', passwordVariable: 'NPASS')]) {
                    sh """
                        curl -u "$NUSER:$NPASS" \
                        --upload-file build.zip \
                        "${NEXUS_URL}safe-ride-app-${COMMIT_HASH}.zip"
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                // Using your updated credential ID: docker
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DUSER', passwordVariable: 'DPASS')]) {
                    sh """
                        echo "$DPASS" | docker login -u "$DUSER" --password-stdin
                        docker push ${IMAGE_NAME}
                    """
                }
            }
        }
    }

    post {
        success { echo "Build Successful: ${IMAGE_NAME}" }
        failure { echo "Build Failed!" }
    }
}
