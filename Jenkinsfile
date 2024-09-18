pipeline {

  agent {
    kubernetes {
      yamlFile 'BuilderPod.yaml'
    }
  }

  parameters {
    booleanParam(name: 'TRIGGER_CD_DSL', defaultValue: false, description: 'Trigger cd-dsl job')
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
