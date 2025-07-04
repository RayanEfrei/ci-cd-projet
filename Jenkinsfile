pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_CREDS = credentials('dockerhub-creds')
        NEXUS_CREDS = credentials('nexus-creds')
        IMAGE_NAME = "rayanefrei/ci-cd-image"
    }

    stages {

        stage('Build Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }

        stage('Scan Sécurité (Trivy)') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH . || true'
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
