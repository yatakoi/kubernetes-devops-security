pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "8a5dfbaa-7d6e-4916-b90d-d6d188c47e5b", url: ""]) {
          sh 'printenv'
          sh 'docker build -t yatakoi/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push yatakoi/numeric-app:""$GIT_COMMIT""'
        }
      }
    }    
  }
}