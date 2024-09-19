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
  }

  stages {
    stage('SonarQube Analysis') {
      steps {
        container('sonar-scanner') {
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
              -Dsonar.sources=. \
              -Dsonar.host.url=${SONAR_HOST_URL} \
              -Dsonar.login=${SONAR_AUTH_TOKEN}
          '''
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
