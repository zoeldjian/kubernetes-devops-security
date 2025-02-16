pipeline {
  agent any
  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "siddharth67/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://devsecops-demo.eastus.cloudapp.azure.com/"
    applicationURI = "/increment/99"
  }

 
  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=testing-application -Dsonar.host.url=http://{IpAddress}:{port} -Dsonar.login=b90cdb6824bbd0ae041826b9cc6d330ae57c78be" 
      }
       timeout(time: 2, unit: 'MINUTES') {
         script {
           waitForQualityGate abortPipeline: true
        }
      }  
    }
  }

    stage('KICS scan') {
            steps {
                script {
                      docker.image('checkmarx/kics:latest').inside("--entrypoint=''") {
                      unstash 'https://github.com/zoeldjian/kubernetes-devops-security'
                      sh('/app/bin/kics scan -p \'\$(pwd)\' -q /app/bin/assets/queries --ci --report-formats html -o \'\$(pwd)\' --ignore-on-exit results')
                      archiveArtifacts(artifacts: 'results.html', fingerprint: true)
                      publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: '.', reportFiles: 'results.html', reportName: 'KICS Results', reportTitles: ''])
                    }
                }
            }
        }

    stage('Vulnerability Scan - Docker') {
      steps {
       parallel (
       "Docker Scan": {
            sh "bash trivy-docker-image-scan.sh"
      },
       "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          }
      )
    }
  }
 
} 

//    stage('Docker Build and Push') {
//      steps {
//        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
//          sh 'printenv'
//          sh 'docker build -t zoeldjian/devsecops-numeric-apps:""$GIT_COMMIT"" .'
//          sh 'docker push zoeldjian/devsecops-numeric-apps:""$GIT_COMMIT""'
//        }
//      }
//    }

//    stage('Kubernetes Deployment - DEV') {
//      steps {
//        withKubeConfig([credentialsId: 'kubeconfig']) {
//          sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
//          sh "kubectl apply -f k8s_deployment_service.yaml"
//        }
//      }
//    }
  }

