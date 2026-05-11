pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "$JAVA_HOME/bin:$PATH"
    }

    stages {
    
    tools{
        jdk 'jdk17'
        maven 'maven2'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    

    stages {
        
        stage('Git Checkout ') {
            steps {
               git branch: 'main', url: 'https://github.com/bhaskar2024958/BoardgameListingWebApp'
            }
        }
        
        stage('Compile') {
            steps {
               sh 'java -version'
               sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
               sh 'mvn test'
            }
        }
        
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=OnlineGame \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=OnlineGame '''
               }
            }
        }
        
     
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven2', mavenSettingsConfig: '', traceability: true) {
                     sh 'mvn deploy'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                    sh "docker build -t javaexpress/boardgame:latest ."
                 }
               }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-fs-report1.html javaexpress/boardgame:latest'
            }
        }
        
        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                    sh "docker push javaexpress/boardgame:latest"
                 }
               }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment-service.yaml'
            }
        }
        
        stage('Verify Deployment') {
            steps {
               sh 'kubectl get pods'
               sh 'kubectl get svc'
            }
        }
    }
}

