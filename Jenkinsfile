pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        NVD_API_KEY = credentials('nvd-api-key')
    }

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {

        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/waghepratiksha21-create/Ekart.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        // Skip failing tests
        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=EKART \
                    -Dsonar.projectName=EKART \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=${NVD_API_KEY}",
                                    odcInstallation: 'DC'
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-maven',
                    jdk: 'jdk17',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh "mvn deploy -DskipTests"
                }
            }
        }

        stage('build and Tag docker image') {
            steps {
                sh "docker build -t waghepratiksha21/ekart:latest -f docker/Dockerfile ."
            }
        }

        stage('Push image to Hub'){
            steps{
                withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                    sh "docker login -u waghepratiksha21 -p ${dockerhubpwd}"
                }
                sh "docker push waghepratiksha21/ekart:latest"
            }
        }

        stage('EKS and Kubectl configuration'){
            steps{
                sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
            }
        }

        stage('Deploy to k8s'){
            steps{
                sh 'kubectl apply -f deploymentservice.yml'
            }
        }
    }
}
