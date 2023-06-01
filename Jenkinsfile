pipeline {   
    agent any
    stages {
        stage("test") {
            steps {
                script{
                    echo "Testing the application...."
                    echo "Testing the integration...."
                }
            }
        }
        stage("build") {
            steps {
                script{
                    echo "Building the application...."
                }
            }
        }
        stage("deploy") {
            steps {
                script{
                    echo "Deploying the application...."
                }
            }
        }               
    }
} 
