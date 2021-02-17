#!groovy

// this option exists to enable jobs to request images be tagged, but to do the
// pushing itself.  It may or may not be used.
properties([
  parameters([
     booleanParam(name: 'PUSH', defaultValue: true, description: 'push tagged images to dockerhub')
  ])
])

node('centos8') {

  checkout scm

  stage('tag') {
    withCredentials([
      usernameColonPassword(
        credentialsId: '0f11d4d1-6557-423c-b5ae-693cc87f7b4b',
        variable: 'HUB_LOGIN'
      )
    ]) {
      echo "push: ${params.PUSH}"

      // set tag based on branch (master == latest)
      def tag = "latest"
      if (env.BRANCH_NAME != 'master') {
        tag = "${env.BRANCH_NAME}"
      }
      echo "using tag: ${tag}"

      // below requires Pipeline Utility Steps plugin in Jenkins
      // https://plugins.jenkins.io/pipeline-utility-steps/
      def versions = readYaml file: 'versions.yml'

      versions.each { image, version ->
        // only pull image if it doesn't exist locally.  This also means that
        // we *won't* pull if it *does* exist locally.  

        sh "if ! [[ \$(podman images ${image}:${version} --format \"{{.ID}}\") ]]; then podman pull --creds \"$HUB_LOGIN\" docker://docker.io/veupathdb/${image}:${version}; fi"
        // should be something here to prevent total job failure for a single problem image/version?

        sh "podman tag ${image}:${version} ${image}:${tag}"
        
        if ( params.PUSH ) {
          sh "echo I will push"
          sh "echo podman push --creds \"$HUB_LOGIN\"  ${image} docker://docker.io/veupathdb/${image}:${tag}"
        }
      }

        
    }
  }
}
