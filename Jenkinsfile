#!/usr/bin/env groovy
@Library('jenkinsEx-shared-library') _

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/OmarRiad/jenkins-shared-library.git',
    credentialsID: 'github-credentials'
    ]
)

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    
    stages {
        stage("increment version"){
        steps{
            script{
                echo "incrementing app version"
                sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'

                def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                def version = matcher[0][1]
                env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                echo "$IMAGE_NAME"
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
                    dockerLogin()
                    dockerBuild("omarriad07/demo-app")
                    dockerPush("omarriad07/demo-app")
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh omarriad07/demo-app:${IMAGE_NAME}"
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
                    sh 'git config --global user.email "jenkins@example.com"'
                    sh 'git config --global user.name "jenkins"'
                    sh "git remote set-url origin https://${GITHUB_API_TOKEN}@github.com/OmarRiad/java-maven-app.git"
                    sh 'git add .'
                    sh 'git commit -m "ci: version bump"'
                    sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
        
    }
}
