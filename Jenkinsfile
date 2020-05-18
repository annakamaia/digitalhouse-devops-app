pipeline {

    agent none

    environment {

        NODE_ENV="development"

        AWS_ACCESS_KEY=""
        AWS_SECRET_ACCESS_KEY=""
        AWS_SDK_LOAD_CONFIG="0"
        REGION="us-east-1" 
        PERMISSION=""
        ACCEPTED_FILE_FORMATS_ARRAY=""

        REGISTRY_ADDRESS = "178955609749.dkr.ecr.us-east-1.amazonaws.com"
                
        CREDENTIALID="awsdevops"
        CREDENTIAL_ECR="ecr:us-east-1:${CREDENTIALID}"
        CREDENTIALID_S3="CredentialsS3"

        VERSION="1.0.0"
    }


    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    triggers {
        cron('@daily')
    }

    stages{
        stage("Build, Test and Push Docker Image") {
            agent {  
                node {
                    label 'master'
                }
            }
            stages {

                stage('Clone repository') {
                    steps {
                        script {
                            if(env.GIT_BRANCH=='origin/dev'){
                                checkout scm
                            }
                            sh('printenv | sort')
                            echo "My branch is: ${env.GIT_BRANCH}"
                        }
                    }
                }
                stage('Build image'){       
                    steps {
                        script {
                            print "Environment will be : ${env.NODE_ENV}"
                            docker.build("pi_gitgirls:latest")
                        }
                    }
                }

                stage('Test image') {
                    steps {
                        script {
                            withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                                , credentialsId: "${CREDENTIALID_S3}"]]) {

                                docker.image("pi_gitgirls:latest").withRun('-p 8030:3000') { c ->
                                    sh 'docker ps'
                                    sh 'sleep 10'
                                    sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'
                                }
                            }
                    
                        }
                    }
                }
                
                stage('Docker push') {
                    steps {
                        echo 'Push latest para AWS ECR'
                        script {
                            docker.withRegistry("https://${REGISTRY_ADDRESS}", "${CREDENTIAL_ECR}") {
                                docker.image('pi_gitgirls').push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Homolog') {
            agent {  
                node {
                    label 'homologacao'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/dev'){
 
                        docker.withRegistry("https://${REGISTRY_ADDRESS}", "${CREDENTIAL_ECR}") {
                            docker.image('pi_gitgirls').pull()
                        }

                        echo 'Deploy para Desenvolvimento'
                        sh "hostname"
                        sh "docker stop app1"
                        sh "docker rm app1"
                        //sh "docker run -d --name app1 -p 8030:3000 178955609749.dkr.ecr.us-east-1.amazonaws.com/pi_gitgirls"
                        withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId: "${CREDENTIALID_S3}"]]) {
                        sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=homolog -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=dh-pigitgirls-homol ${REGISTRY_ADDRESS}/pi_gitgirls:latest"
                        }
                        
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

        stage('Deploy to Producao') {
            agent {  
                node {
                    label 'producao'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/prod'){
 
                        environment {

                            NODE_ENV="production"
                            AWS_ACCESS_KEY=${AWS_ACCESS_KEY_ID}
                            AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                            AWS_SDK_LOAD_CONFIG="0"
                            BUCKET_NAME="dh-pigitgirls-prod"
                            REGION="us-east-1" 
                            PERMISSION=""
                            ACCEPTED_FILE_FORMATS_ARRAY=""
                        }


                        docker.withRegistry("https://${REGISTRY_ADDRESS}", "${CREDENTIAL_ECR}") {
                            docker.image('pi_gitgirls').pull()
                        }

                        echo 'Deploy para Producao'
                        sh "hostname"
                        sh "docker stop app1"
                        sh "docker rm app1"
                        //sh "docker run -d --name app1 -p 8030:3000 178955609749.dkr.ecr.us-east-1.amazonaws.com/pi_gitgirls:latest"
                        withCredentials([[$class:'AmazonWebServicesCredentialsBinding' 
                            , credentialsId: "${CREDENTIALID_S3}"]]) {
                          sh "docker run -d --name app1 -p 8030:3000 -e NODE_ENV=producao -e AWS_ACCESS_KEY=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e BUCKET_NAME=dh-pigitgirls-homol ${REGISTRY_ADDRESS}/pi_gitgirls:latest"
                        }
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:8030/api/v1/healthcheck'

                    }
                }
            }

        }

    }
}
