pipeline {

    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sonarqube')          // SONAR TOKEN
        DOCKERHUB_CRED = credentials('docker')              // DOCKERHUB USER & PASS
        NEXUS_CRED = credentials('nexus')                   // NEXUS USER & PASS

        SONAR_HOST_URL = "http://18.117.125.177:9000"       // YOUR SONAR URL

        COMMIT_HASH = "${env.GIT_COMMIT[0..6]}"             // SAFE WAY
        IMAGE_NAME = "rishitha2707/safe-ride-app:${COMMIT_HASH}"

        NEXUS_URL = "http://52.8.253.11:8081/repository/npm/"
    }

    tools {
        nodejs 'Node16'     // Ensure Jenkins tool name is EXACT
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Commit: ${env.GIT_COMMIT}"
                }
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
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=my-node-app \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=node_modules/**,build/** \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.token=${SONARQUBE_TOKEN}
                    """
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
                    rm -f build.zip
                    zip -r build.zip build/
                """
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    curl -v -u ${NEXUS_CRED_USR}:${NEXUS_CRED_PSW} \
                    --upload-file build.zip \
                    ${NEXUS_URL}safe-ride-app-${COMMIT_HASH}.zip
                """
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
                sh """
                    echo "${DOCKERHUB_CRED_PSW}" | docker login -u "${DOCKERHUB_CRED_USR}" --password-stdin
                    docker push ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "Build Successful: ${IMAGE_NAME}"
        }
        failure {
            echo "Build Failed!"
        }
    }
}
