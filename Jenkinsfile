pipeline {
    agent any

    tools {
        maven 'maven3' // Assure-toi que ce nom est bien celui configuré dans Jenkins
    }

    environment {
        DOCKER_CREDS = credentials('dockerhub-creds') // Ton identifiant DockerHub
        NEXUS_CREDS = credentials('nexus-creds')       // Ton identifiant Nexus
        IMAGE_NAME = "rayanefrei/ci-cd-image"          // Change si ton image a un autre nom
    }

    stages {
        stage('Build Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }

        stage('Push Artefact Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'NEXUS_PASS', usernameVariable: 'NEXUS_USER')]) {
                    sh '''
                        mvn deploy -DaltDeploymentRepository=internal.repo::default::http://192.168.1.150:8081/repository/maven-releases \
                        -Dnexus.username=$NEXUS_USER -Dnexus.password=$NEXUS_PASS
                    '''
                }
            }
        }
    }
}
