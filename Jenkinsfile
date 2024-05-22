#!/usr/bin/env groovy

@Library('jenkinsEx-shared-library') _


pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    
    stages {
        stage("increment version"){
        steps{
            script{
                 IncrementV()
            }
        }
    }
        
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@13.38.11.113"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }               
        }
        stage("commit version update"){
      steps{
        script{
          withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_API_TOKEN')]){
                        setUser()
                        gitPush()
                    }
        }
      }
    }
        
    }
}
