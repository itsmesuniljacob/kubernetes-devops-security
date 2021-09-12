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
  
    //  stage('SonarQube - SAST') {
    //   steps {
    //     sh "mvn sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://devsecops314.eastus.cloudapp.azure.com:9000 -Dsonar.login=2a05c238ff67f0211ca5b3685a8bf44b748eaa7e"
    //   }
    // }
    stage('Vulnerability Scan - Docker ') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash trivy-docker-image-scan.sh"
          },
          "OPA Conftest": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
          }
        )
      }
    }
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub",url: ""]) {
          sh 'printenv'
          sh 'sudo docker build -t xylene1980/numeric-app:""$GIT_COMMIT"" .'
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
  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
//      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
  }
    // success {

    // }

    // failure {

    // }
  }
}