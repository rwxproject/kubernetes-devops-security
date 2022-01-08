pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
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

//     stage('Mutation Tests - PIT') {
//       steps {
//         sh "mvn org.pitest:pitest-maven:mutationCoverage"
//       }
//       post {
//         always {
//           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
//         }
//       }
//     }
    
//     stage('SonarQube - SAST') {
//       steps {
//         sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecopssb.eastus.cloudapp.azure.com:9000 -Dsonar.login=69e1b4c1f6f3bdaba69dbb0012b65872ec124348"
//       }
//     }

    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric -Dsonar.host.url=http://devsecopssb.eastus.cloudapp.azure.com:9000"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
    
    
    
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t rwxproject/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push rwxproject/numeric-app:""$GIT_COMMIT""'
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
