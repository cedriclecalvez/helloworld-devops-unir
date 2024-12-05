pipeline {
  agent any
  stages {
    stage('checkout-repo') {
      steps {
        git(url: 'https://github.com/cedriclecalvez/helloworld-devops-unir', branch: 'master')
      }
    }

  }
}