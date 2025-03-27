pipeline {
    agent any
    
     tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
      
     
        stage('Git Checkout ') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Arshad-Sk/Multitier-bankwebapp.git'
            }
        }
        
          stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        
         stage('Unit Tests') {
            steps {
                    sh "mvn test -DskipTests=true"
            }
        }
        
          stage("TRIVY FS Scan"){
            steps{
                sh "trivy  fs --format table  -o fs-report.html ."
            }
        }
        
         stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=MultitierBankApp \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=MultitierBankApp '''
    
                }
            }
        }
        
        
    
           stage("Maven Build"){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
        
             stage("Push to Nexus"){
            steps{
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh "mvn deploy -DskipTests=true"
               }
            }
        }
        
        stage("Docker Build and Tag"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'b9f5bd82-bfa9-4e28-a440-6792e37b103e', toolName: 'docker') {
                        
                        sh "docker build -t sk77arshad/bankapp:latest ."
                     
                                           }
                }
            }
        }
        
            stage("TRIVY Image Scan"){
            steps{
                sh "trivy  image --format table -o fs-report.html sk77arshad/bankapp:latest"
            }
        }
        
          stage("Docker Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'b9f5bd82-bfa9-4e28-a440-6792e37b103e', toolName: 'docker') {
                            sh "docker push sk77arshad/bankapp:latest"
                                                          }
                }
            }
        }
        
        
            stage('Deplye to K8') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'my-eks-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://7AED98DC14C13FC317CD2487E33A9339.gr7.eu-north-1.eks.amazonaws.com') {
               sh "kubectl apply -f ds.yml -n webapps"
               sleep 30
}
            }
        }
    
                stage('Verify K8 Deployment ') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'my-eks-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://7AED98DC14C13FC317CD2487E33A9339.gr7.eu-north-1.eks.amazonaws.com') {
               sh "kubectl get pods -n webapps"
               sh "kubectl get svc -n webapps"
             }
            }
        }
        
}
}
