pipeline {
    agent any
    environment {
        registry = "venkatesheg12/python-app"
        registryCredential = 'dockerhub'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'project-1', url: 'https://github.com/venkateshgidugu/upgrad-project1.git'
            }
        }
        
        stage('Build docker image') {
            steps {
                script {
                  dockerImage = docker.build registry + ":v$BUILD_NUMBER"
              }
            }
        }
        stage('Upload Image'){
          steps{
            script {
              docker.withRegistry('', registryCredential) {
                dockerImage.push("v$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:v$BUILD_NUMBER"
          }
        }
        stage('Deploying container to Kubernetes') {
            steps {
                script {
                    def serviceExists = ""
                   serviceExists = sh(script: "kubectl get services python-app -n default | grep python-app | awk '{ print \$1}'", returnStdout: true).trim()
                    echo serviceExists 
                    if (serviceExists == "python-app" ) {
                         sh "echo 'Upgrading...'"
                        sh "helm upgrade project-1 python-project --set appimage=${registry}:v${BUILD_NUMBER}"
                    
                    } else {
                         sh "echo 'Installing...'"
                         sh "helm install project-1 python-project --set appimage=${registry}:v${BUILD_NUMBER}"
                        }
               }
            }
        }    
        stage('Monitoring with Prometheus & Grafana') {
           steps {
                sh "helm repo add prometheus-community https://prometheus-community.github.io/helm-charts"
                sh "helm repo update"
                sh "helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack"
            }
        }
        

    }
}