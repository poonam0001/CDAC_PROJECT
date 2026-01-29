pipeline {
    agent { label 'docker-agent' }

    environment {
        IMAGE_NAME = "poonamyadav361/flask-blog"
        IMAGE_TAG  = "latest"
        SONAR_HOME = tool 'sonar'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/poonam0001/CDAC_PROJECT.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=flask_blog \
                        -Dsonar.projectKey=flask_blog
                    """
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "/usr/bin/docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "${DOCKER_PASS}" | /usr/bin/docker login -u "${DOCKER_USER}" --password-stdin
                        /usr/bin/docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    /usr/local/bin/docker-compose pull
                    /usr/local/bin/docker-compose up -d --force-recreate
                """
            }
        }
    }
}
