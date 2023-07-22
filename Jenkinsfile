pipeline {
    agent any

    stages {
        stage("git code") {
            steps {
                echo "cloning code"
                git url: "https://github.com/adguchiya/noteapp-django.git", branch: "main"
            }
        }

        stage("build") {
            steps {
                echo "building a docker image"
                sh "docker build -t notebook:latest ."
            }
        }

        stage("dockerhub login") {
            steps {
                echo "dockerhub login"
                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerhub",
                        usernameVariable: "username",
                        passwordVariable: "password"
                    )
                ]) {
                    sh "docker login -u $username -p $password"
                }
            }
        }

        stage("pushing code to dockerhub") {
            steps {
                echo "pushing image to docker hub"
                sh "docker tag notebook:latest $username/notebook:latest"
                sh "docker push $username/notebook:latest"
            }
        }

        stage("deploy") {
            steps {
                echo "deploying image as a docker container"
                sh "docker-compose down"
                sh "docker-compose up -d"
            }
        }
    }
}
