pipeline {
    agent {
        label 'k8s-slave'
    }

    tools {
        maven 'maven-3.8.8'
        jdk 'jdk-17'
    }

    environment {
        APPLICATION_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        //DOCKER REPO INFO
        DOCKER_HUB = "docker.io/kishoresamala84"
        DOCKER_CREDS = credentials('kishoresamala84_docker_creds')
        //DOCKER VM INFO
        DOCKER_VM_IP = "35.245.49.208"
        // JOHN_CREDS = credentials('john_docker_vm_creds')
    }

    stages {
        stage ('BUILD_STAGE') {
            steps {
                script {
                    echo " ***** Maven Build Stage ***** "
                    sh "mvn clean package -DskipTest=true"
                    archiveArtifacts 'target/*.jar'
                }
            }
        }

        stage ('SONARQUBE_SCANNER') {
            steps {
                withSonarQubeEnv('sonarqube'){
                    sh """
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=i27-eureka-06 \
                        -Dsonar.host.url=http://34.21.15.97:9000 \
                        -Dsonar.login=sqa_0acf0f3c8f99b362f8d7b4ce6c7d9ec8d26db415
                    """
                }
                timeout(time: 2, unit: 'MINUTES'){
                waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ('BUILD_FORMAT') {
            steps {
                script {
                    sh "echo Source JAR format: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    sh "echo Destination JAR format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                }
            }
        }

        stage ('DOCKER_BUILD_AND_PUSH') {
            steps {
                script {
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd"
                    sh "docker login -u ${env.DOCKER_CREDS_USR} -p ${env.DOCKER_CREDS_PSW}"
                    sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                }
            }
        }

        stage ('DEPLOY_TO_DOCKER') {
            steps {
                echo " ***** Deploying to DEV env ***** "
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        try {
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$DOCKER_VM_IP \"docker stop ${env.APPLICATION_NAME}-dev\""
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$DOCKER_VM_IP \"docker rm ${env.APPLICATION_NAME}-dev\""
                        }
                        catch (err) {
                            echo "Error Caught: $err"
                        }
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$DOCKER_VM_IP \"docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME}-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}\""
                        // sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@'$env.DOCKER_VM_IP' \"docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME}-dev${env.DOCKER_HUB}:${GIT_COMMIT}\""
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def Subject = "JOB IS SUCCESS !!! Job Name is =:-> [${env.JOB_NAME}] <<>> Build Number # is =:-> [${env.BUILD_NUMBER}] <<>> Status is =:-> [${currentBuild.currentResult}]"
                def Body = "Job URL :=> ${env.BUILD_URL} \n\n" +
                        "Build_Number is :=> ${env.BUILD_NUMBER} \n\n" +
                        "Build_Status is :=> ${currentBuild.currentResult}"
                sendEmailNotification('kishorecloud.1725@gmail.com', subject, body)            
            } 
        }

        failure {
            script {
                def Subject = "JOB FAILED !!! Job Name is =:-> [${env.JOB_NAME}] <<>> Build Number # is =:-> [${env.BUILD_NUMBER}] <<>> Status is =:-> [${currentBuild.currentResult}]"
                def Body = "Job URL :=> ${env.BUILD_URL} \n\n" +
                        "Build_Number is :=> ${env.BUILD_NUMBER} \n\n" +
                        "Build_Status is :=> ${currentBuild.currentResult}"
                sendEmailNotification('kishorecloud.1725@gmail.com', subject, body)            
            }             
        }
    }
}

/// METHODS ///

 def sendEmailNotification(String recipient, String subject, String body) {
    mail (
        to: recipient,
        subject: subject,
        body: body
    )
}
