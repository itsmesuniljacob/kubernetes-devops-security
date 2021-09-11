pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }
      stage('Unit test') {
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

    //   stage('Mutation Tests - PIT') {
    //     steps {
    //       sh "mvn org.pitest:pitest-maven:mutationCoverage"
    //     }
    //   post {
    //     always {
    //       pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    //     }
    //   }
    // }
  
     stage('SonarQube - SAST') {
      steps {
        sh "mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://devsecops314.eastus.cloudapp.azure.com:9000 -Dsonar.login=63b061bca77df5df8ff202838547ca6959157246"
      }
    }
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub",url: ""]) {
          sh 'printenv'
          sh 'docker build -t xylene1980/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push xylene1980/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#xylene1980/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}