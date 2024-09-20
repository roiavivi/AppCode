pipeline {
  agent {
    kubernetes {
      yamlFile 'BuilderPod.yaml'
    }
  }

  parameters {
    booleanParam(name: 'TRIGGER_CD_DSL', defaultValue: false, description: 'Trigger cd-dsl job')
  }

  environment {
    SONAR_HOST_URL = 'https://sonarqube.roiavivi.com'
    SONAR_PROJECT_KEY = 'app'
    // Add your SonarQube token as a secret text credential in Jenkins and reference it here
    SONAR_AUTH_TOKEN = credentials('sonar-auth-token') // Replace 'sonar-auth-token' with your actual credential ID
    REPOSITORY_URI = 'your-repository-uri/' // Replace with your actual repository URI
    AWS_ECR_REPO_NAME = 'your-ecr-repo-name' // Replace with your actual ECR repository name
  }

  // stages {
  //   stage('SonarQube Analysis') {
  //     steps {
  //       container('sonar-scanner') {
  //         sh '''
  //           sonar-scanner \
  //             -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
  //             -Dsonar.sources=. \
  //             -Dsonar.host.url=${SONAR_HOST_URL} \
  //             -Dsonar.login=${SONAR_AUTH_TOKEN}
  //         '''
  //       }
  //     }
  //   }

  //   stage('Quality Check') {
  //     steps {
  //       container('sonar-scanner') {
  //         script {
  //           def projectStatus = sh(
  //             script: '''
  //               status=$(curl -s -u ${SONAR_AUTH_TOKEN}: ${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY} | jq -r '.projectStatus.status')
  //               if [ "$status" != "OK" ]; then
  //                 echo "Quality gate failed: $status"
  //                 exit 1
  //               else
  //                 echo "Quality gate passed: $status"
  //               fi
  //             ''',
  //             returnStatus: true
  //           )
  //           if (projectStatus != 0) {
  //             error "Quality gate failed"
  //           }
  //         }
  //       }
  //     }
  //   }

    stage('Trivy File Scan') {
      steps {
        container('trivy') {
          sh 'trivy fs . > trivyfs.txt'
        }
      }
    }

    stage('Build with Kaniko') {
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
          sh '''#!/busybox/sh
            /kaniko/executor --context `pwd` --dockerfile `pwd`/Dockerfile --destination roie710/app:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('TRIVY Image Scan') {
      steps {
        container('trivy') {
          sh 'trivy image roie710/app:${BUILD_NUMBER} > trivyimage.txt'
        }
      }
    }

    stage('Trigger CD-DSL job') {
      when {
        expression { return params.TRIGGER_CD_DSL }
      }
      steps {
        build job: 'CD-DSL', 
              parameters: [
                string(name: 'BUILD_NUMBER', value: "${BUILD_NUMBER}")
              ],
              wait: true
      }
    }
  }
}
