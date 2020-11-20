#!groovy

node('centos8') {
  def tag = "latest"
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

      // run tagging script

      // tag versions.yml ${env.BRANCH_NAME}

      def versions = readYaml file: 'versions.yml'

      versions.each { image, version ->
        sh "echo podman pull --creds \"$HUB_LOGIN\" veupathdb/${image}:${version}"
        sh "echo podman tag ${image}:${version} ${image}:${env.BRANCH_NAME}"
        sh "echo podman push --creds \"$HUB_LOGIN\"  ${image} docker://docker.io/veupathdb/${image}"
      }

        
      // sh "echo ${versions}"
      // push to dockerhub (for now)
      // sh "podman push --creds \"$HUB_LOGIN\" ${imageName} docker://docker.io/veupathdb/${imageName}:${tag}"
    }
  }
}
