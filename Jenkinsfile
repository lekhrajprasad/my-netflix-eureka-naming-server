def imageName = "lekhrajprasad/my-eureka-server"
def gitBranch = "master"
def gitUrl = "https://github.com/lekhrajprasad/my-netflix-eureka-naming-server.git"
pipeline {
agent any
options {
timeout(time: 50, unit: 'MINUTES')
ansiColor('xterm')
buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
}
environment {
// Default tool options breaks the pipeline. Removed misbehaving option (Unrecognized VM option 'UseCGroupMemoryLimitForHeap').
JAVA_TOOL_OPTIONS = '-XX:+UnlockExperimentalVMOptions -Dsun.zip.disableMemoryMapping=true'
}
stages {
stage('GitClone') {
steps {
script {
checkout([$class: 'GitSCM', branches: [[name: "${gitBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'gitaccess', url: "${gitUrl}"]]])
}
}
}
stage('Versioning') {
steps {
script {
version_parser = readMavenPom file: './pom.xml'
version = "${version_parser.version}"
}
}
}
stage('Build Image') {
steps {
script {
sh """
docker build -t "${imageName}" .
"""
}
}
}
stage('Re-tag image') {
steps {
script {
sh """
docker tag "${imageName}" "${imageName}:${version}"
"""
}
}
}
stage('Image Push to Dockerhub') {
steps {
script {
// This step should not normally be used in your script. Consult the inline help for details.
withDockerRegistry(credentialsId: 'dockerhub') {
sh """
docker push "${imageName}"
docker push "${imageName}:${version}"
"""
}
}
}
}
}
}
