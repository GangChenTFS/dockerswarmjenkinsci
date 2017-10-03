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
            ./cd/publish.sh alpha dtr-dell.caleb.boxboat.net $version
            export DOCKER_TLS_VERIFY=1
            export DOCKER_CERT_PATH=/home/ubuntu/ucp-bundle
            export DOCKER_HOST=tcp://ucp-dell.caleb.boxboat.net:443
            ./cd/deploy-swarm.sh alpha dtr-dell.caleb.boxboat.net $version
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
