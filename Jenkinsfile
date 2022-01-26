pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later..!!
            }
        }

      stage('Sonarqube - SAST') {
            steps {
              sh "sonar-scanner -Dsonar.projectKey=testing-application -Dsonar.sources=. -Dsonar.host.url=http://18.141.173.8:9000 -Dsonar.login=fbdacdf7e2ac3c7f4731eae6c4116fb5cc58be56"
              
            }
        }
    }
}











