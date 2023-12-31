pipeline {
    agent any

    environment {
        SONARQUBE_URL = "http://172.17.0.1:9000" // Change this URL if SonarQube is hosted elsewhere
        SONAR_PROJECT_KEY = "notebook"
        SONAR_AUTH_TOKEN = "sqa_7cdbd4c459d60f6ee360c2dde16bc566dca70ff4"
        YOUR_REPO = "/var/lib/jenkins/workspace/notebook" // Update this with the actual path to your project directory on the Jenkins agent
    }

    stages {
        stage("git code") {
            steps {
                echo "Cloning code"
                git url: "https://github.com/adguchiya/noteapp-django.git", branch: "main"
            }
        }

        stage("sonarqube analysis") {
            steps {
                echo "Running SonarQube analysis with Docker"

                script {
                    try {
                        // Run the SonarQube Scanner analysis in a Docker container
                        sh "docker run --rm -e SONAR_HOST_URL=${SONARQUBE_URL} -e SONAR_SCANNER_OPTS=-Dsonar.projectKey=${SONAR_PROJECT_KEY} -e SONAR_TOKEN=${SONAR_AUTH_TOKEN} -v ${YOUR_REPO}:/usr/src sonarsource/sonar-scanner-cli"
                    } catch (Exception sonarError) {
                        echo "SonarQube analysis failed!"
                        throw sonarError
                    }
                }
            }
        }

        stage("build") {
            steps {
                echo "Building a docker image"
                sh "docker build -t notebook:latest ."
            }
        }

        stage("dockerhub login") {
            steps {
                echo "Dockerhub login"
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerhub",
                        usernameVariable: "username",
                        passwordVariable: "password"
                    )
                ]) {
                    sh "docker login -u $username -p $password"
                    sh "docker tag notebook:latest $username/notebook:latest"
                    sh "docker push $username/notebook:latest"
                }
            }
        }

        stage("deploy") {
            steps {
                echo "Deploying image as a docker container"
                sh "docker-compose down"
                sh "docker-compose up -d"
            }
        }
    }
}
