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
// splunkpath="${config.splunkurl}"
REPOSITORY_BRANCH="${PULL_REQUEST_FROM_BRANCH}"
// COMPONENT="${config.serviceName}"
}
stages {
stage('GitClone') {
steps {
script {
String payload = "${payload}"
jsonObject = readJSON text: payload
gitHash = "${jsonObject.pull_request.head.sha}"
String gitUrl = "${jsonObject.repository.clone_url}"
checkout([$class: 'GitSCM', branches: [[name: "${gitHash}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'gitaccess', url: "${gitUrl}"]]])
}
}
}
stage('Get Dependencies') {
steps {
script {
sh """
mvn clean compile
"""
}
}
}
stage('Unit Test') {
steps {
script {
sh """
mvn test
"""
}
}
}
stage('CodeQuality') {
steps {
script {
// -Dsonar.login=9d0d4dbbe11657dfb68c5d73dab4884176b3a72c
withSonarQubeEnv(credentialsId: 'sonar-access-token') {
// some block
sh """
sonar-scanner \
-Dsonar.projectKey="my-pr" \
-Dsonar.java.binaries="./target/classes" \
-Dsonar.sources="." \
-Dsonar.host.url="http://54.175.47.141" \
-Dsonar.branch.target="master"
"""
}
}
}
}
stage("Quality Gate") {
options {
timeout(time: 10, unit: 'MINUTES')
}
steps {
sleep(60)
// Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
// true = set pipeline to UNSTABLE, false = don't
waitForQualityGate abortPipeline: true
}
}
}
}
