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
                // Tell Jenkins to switch to the smswebapp directory
                dir('smswebapp') {
                    script {
                        // bat "docker build -t mysmswebapp:latest ."
                        bat "docker build -f smswebapp/Dockerfile -t mysmswebapp:latest ."
                    }
                }
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                script {
                    // Securely inject the DockerHub credentials into the environment
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        
                        
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        
                       
                        bat 'docker tag myapp:latest %DOCKER_USER%/myapp:latest'
                        
                       
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
