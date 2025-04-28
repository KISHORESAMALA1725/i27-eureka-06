pipeline {
    agent {
        label 'k8s-slave'
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
            script {
                echo " ***** Maven Build Stage ***** "
                sh "mvn clean package -DskipTest=true"
                archiveArtifacts 'target/*.jar'
            }
        }

        
    }
}