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

    stage('Mutation Tests - PIT') {
          steps {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
          post {
            always {
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
          }
        }

    stage('SonarQube - SAST') {
      steps {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://jenkins.max-ko.ru:9000 -Dsonar.token=sqp_e73ee6c04b2d8ccaa7f89d0e90ce6d9655c87499"
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t yatakoi/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push yatakoi/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage('Kubernetes Deployment - DEV') {
      steps {
//        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#yatakoi/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        //}
      }
    }
  }
}