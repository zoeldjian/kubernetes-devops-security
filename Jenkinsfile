pipeline {
  agent any

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
        sh "sonar-scanner Dsonar.projectKey=sirka-new-landing-page Dsonar.sources=. Dsonar.host.url=http://172.16.2.0:9000 Dsonar.login=dc9703ca7a29795ce14d3bb48971d33e982a6192"
      }
       timeout(time: 2, unit: 'MINUTES') {
         script {
           waitForQualityGate abortPipeline: true
        }
      }  
    }
  }

    stage('Vulnerability Scan - Docker') {
      steps {
            sh "bash trivy-docker-image-scan.sh"
      }
   } 

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t zoeldjian/devsecops-numeric-apps:""$GIT_COMMIT"" .'
          sh 'docker push zoeldjian/devsecops-numeric-apps:""$GIT_COMMIT""'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}

