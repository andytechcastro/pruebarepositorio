pipeline {
  agent any
  environment {
    WORKINGDIR = "./"
  }
  stages {
    stage('Configure Kubernetes Access') {
      steps {
        sh """
        aws eks update-kubeconfig --name continuous-testing --alias continuous-testing
        """
      }
    }
    stage('Deploy QA') {
      when {
        beforeAgent true
        allOf {
          changeRequest target: 'master'
        }
      }
      steps{
        dir(WORKINGDIR) {
          sh """
          helm upgrade andres demo/ --set persistence.enabled=false --debug --wait --install --namespace qa
          """
          script {
            rc = sh(script: "helm test andres --logs --namespace qa", returnStatus: true)
            if (rc != 0) {
              sh """
              helm rollback andres 0 -n qa
              exit 1
              """
            }
          }
          sh """
           kubectl delete pod -n qa andres-demo-test-connection
          """
        }
      }
    }
    stage('Deploy Prod') {
      when {
        beforeAgent true
        allOf {
          branch 'master'
        }
      }
      steps{
        dir(WORKINGDIR) {
          sh """
          helm upgrade andres demo/ --set persistence.enabled=false --debug --wait --install --namespace prod
          """
          script {
            rc = sh(script: "helm test andres --logs --namespace prod", returnStatus: true)
            if (rc != 0) {
              sh """
              helm rollback andres 0 -n prod
              exit 1
              """
            }
          }
          sh """
           kubectl delete pod -n prod andres-demo-test-connection
          """
        }
      }
    }
  }

}
