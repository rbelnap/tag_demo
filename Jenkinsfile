#!groovy

properties([
  parameters([
     booleanParam(name: 'PUSH', defaultValue: true, description: 'push tagged images to dockerhub')
  ])
])

node('centos8') {

  def gitUrl = 'https://github.com/rbelnap/tag_demo.git'

  stage('checkout') {
    checkout([
      $class: 'GitSCM',
      branches: [[name: env.BRANCH_NAME ]],
      doGenerateSubmoduleConfigurations: false,
      extensions: [[
                     $class: 'SubmoduleOption',
                     disableSubModules: true
                   ]],
      userRemoteConfigs:[[url: gitUrl]]
    ])
  }

  stage('tag') {
    withCredentials([
      usernameColonPassword(
        credentialsId: '0f11d4d1-6557-423c-b5ae-693cc87f7b4b',
        variable: 'HUB_LOGIN'
      )
    ]) {
echo "push: ${params.PUSH}"

      // below requires Pipeline Utility Steps plugin in Jenkins
      // https://plugins.jenkins.io/pipeline-utility-steps/
      def versions = readYaml file: 'versions.yml'

      versions.each { image, version ->
        // only pull image if it doesn't exist locally.  This also means that
        // we *won't* pull if it *does* exist locally.  

        sh "if ! [[ \$(podman images ${image}:${version} --format \"{{.ID}}\") ]]; then podman pull --creds \"$HUB_LOGIN\" docker://docker.io/veupathdb/${image}:${version}; fi"
        // should be something here to prevent total job failure for a single problem image/version?

        sh "podman tag ${image}:${version} ${image}:${env.BRANCH_NAME}"
        
        if ( params.PUSH ) {
          sh "echo I will push"
          sh "echo podman push --creds \"$HUB_LOGIN\"  ${image} docker://docker.io/veupathdb/${image}:${env.BRANCH_NAME}"
        }
      }

        
    }
  }
}
