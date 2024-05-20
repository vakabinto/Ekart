pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('GIT CHECKOUT') {
            steps {
                git branch: 'main', url: 'https://github.com/vakabinto/Ekart.git'
            }
        }
        stage('CODE COMPILE') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('CODE TEST') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('SONAR CODE ANALYSIS') {
            steps {
                withSonarQubeEnv( 'sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                             -Dsonar.projectKey=EKART \
                             -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('MVN BUILD') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('DEPLOY TO NEXUS') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global_settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('DOCKER BUILD IMAGE') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                       sh " docker build -t vakabinto/e-kart:latest -f docker/Dockerfile . "
                   }
                }
            }
        }
        stage('TRIVY IMAGE SCAN') {
            steps {
                sh ' trivy image vakabinto/e-kart:latest > trivy-report.txt'
            }
        }
        stage('DOCKER PUSH IMAGE') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                       sh " docker push vakabinto/e-kart:latest "
                   }
                }
            }
        }
        stage('KUBERNETES DEPLOY') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', serverUrl: 'https://172.31.24.94:6443']]) {
                    sh " kubectl apply -f deploymentservice.yml -n webapps"
                     sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
