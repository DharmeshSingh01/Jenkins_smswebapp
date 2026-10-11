pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "C:\\Program Files\\dotnet"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Restoring dependencies
                    //bat "cd ${DOTNET_CLI_HOME} && dotnet restore"
                    bat "dotnet restore"

                    // Building the application
                    bat "dotnet build --configuration Release"
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    // Build the initial local image
                    bat "docker build -t mysmswebapp:latest ."
                }
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                script {
                    // Securely inject the DockerHub credentials into the environment
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        
                        // 1. Log into DockerHub. 
                        // We use echo and --password-stdin so your password is never written to the Jenkins build logs.
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        
                        // 2. Tag the image with your DockerHub username so Docker knows where to push it
                        bat 'docker tag myapp:latest %DOCKER_USER%/myapp:latest'
                        
                        // 3. Push the image to your DockerHub repository
                        bat 'docker push %DOCKER_USER%/myapp:latest'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Running tests
                    bat "dotnet test --no-restore --configuration Release"
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    // Publishing the application
                    bat "dotnet publish --no-restore --configuration Release --output .\\publish"
                }
            }
        }
    }

    post {
        success {
            echo 'Build, test, and publish successful!'
        }
    }
}
