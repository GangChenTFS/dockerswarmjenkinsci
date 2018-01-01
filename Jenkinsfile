throttle(['throttleDocker']) {
  node('docker') {
    wrap([$class: 'AnsiColorBuildWrapper']) {
      try{
        stage('Setup') {
          checkout scm
          sh '''
            ./ci/docker-down.sh
            ./ci/docker-up.sh
          '''
        }
        stage('Test'){
          parallel (
            "unit": {
              sh '''
                ./ci/test/unit.sh
              '''
            },
            "functional": {
              sh '''
                ./ci/test/functional.sh
              '''
            }
          )
        }
        stage('Capacity Test') {
          sh '''
            ./ci/test/stress.sh
          '''
        }
        stage('Deploy to Docker Swarm') {
          sh '''
            version=$(date +%Y%m%d%H%M)
            ./cd/publish.sh alpha dtr.caleb.boxboat.net $version
            export DOCKER_HOST="tcp://192.168.1.101:2375"
            ./cd/deploy-swarm.sh alpha dtr.caleb.boxboat.net $version
          '''
        }
      }
      finally {
        stage('Cleanup') {
          sh '''
            ./ci/docker-down.sh
          '''
        }
      }
    }
  }
}
