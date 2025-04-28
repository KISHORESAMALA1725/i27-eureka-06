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
        DOCKER_HUB = "docker.io/kishoresamala84"
        DOCKER_CREDS = credentials('kishoresamala_docker_creds')
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
                    sh "Source JAR format: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    sh "Destination JAR format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                }
            }
        }
    }
}