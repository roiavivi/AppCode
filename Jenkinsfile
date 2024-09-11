pipeline {

  agent {
    kubernetes {
      yamlFile 'BuilderPod.yaml'
    }
  }

  parameters {
    booleanParam(name: 'TRIGGER_APP_CD', defaultValue: false, description: 'Trigger app-cd job')
  }

  stages {
    stage('Test image') {
      steps {
        sh 'ls -l'
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

    stage('Trigger app-cd job') {
      when {
        expression { return params.TRIGGER_APP_CD }
      }
      steps {
        build job: 'App-CD', 
              parameters: [
                string(name: 'BUILD_NUMBER', value: "${BUILD_NUMBER}")
              ],
              wait: true
      }
    }
  }
}
