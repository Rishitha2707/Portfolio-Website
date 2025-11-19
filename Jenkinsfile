pipeline {

    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sonar')
        DOCKERHUB = credentials('docker')
        NEXUS = credentials('nexus')

        COMMIT_HASH = "${GIT_COMMIT.substring(0,7)}"
        IMAGE_NAME = "yourdockerhubusername/safe-ride-app:${COMMIT_HASH}"

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
                withSonarQubeEnv('Sonar') {      // Your SonarQube server name
                    sh """
                    sonar-scanner \
                      -Dsonar.projectKey=my-node-app \
                      -Dsonar.sources=. \
                      -Dsonar.exclusions=node_modules/**,build/** \
                      -Dsonar.host.url=${SONAR_HOST_URL} \
                      -Dsonar.login=${SONARQUBE_TOKEN}
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
                curl -v -u ${NEXUS_USR}:${NEXUS_PSW} \
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
                echo "${DOCKERHUB_PSW}" | docker login -u "${DOCKERHUB_USR}" --password-stdin
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
