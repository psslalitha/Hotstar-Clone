pipeline {
    agent any
    tools{
        jdk 'java'
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME=tool 'sonarqube'
    }

    stages {
        stage('workspace') {
            steps {
                cleanWs()
            }
        }
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/psslalitha/Hotstar-Clone.git'
            }
        }
        stage('sonarqube-analysis') {
            steps {
                withSonarQubeEnv("sonar-server"){
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=hotstar \
                    -Dsonar.projectKey=hotstar"
                }
            }
        }
        stage('quality-gateway') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 's-id'
                }
            }
        }
        stage("npm install"){
            steps{
                sh 'npm install'
            }
        }
        stage("OWASP"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'

            }
        }
        
        stage('Docker Scout FS') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'd-id', toolName: 'docker'){
                       sh 'docker-scout quickview fs://.'
                       sh 'docker-scout cves fs://.'
                   }
                }   
            }
        
        }
        stage("Docker Build and Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'd-id', toolName: 'docker'){
                       sh "docker build -t hotstar1 ."
                       sh "docker tag hotstar1 srilalithac/hotstar:latest "
                       sh "docker push srilalithac/hotstar:latest"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'd-id', toolName: 'docker'){
                       sh 'docker-scout quickview srilalithac/hotstar:latest'
                       sh 'docker-scout cves srilalithac/hotstar:latest'
                       sh 'docker-scout recommendations srilalithac/hotstar:latest'
                   }
                }   
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('K8S') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-id', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }   
                    }
                }
            }
        }
    }
}
