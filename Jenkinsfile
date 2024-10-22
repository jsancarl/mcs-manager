import java.text.SimpleDateFormat

pipeline {
    agent {
        kubernetes {
            cloud 'minikube'
            inheritFrom 'jenkins-agent'
            defaultContainer 'execution'
        }
    }

    environment {
        KUBE_CONFIG = credentials('kubeconfig')
    }

    stages {
        stage("Install utils") {
            steps {
                sh "apk update"
                sh "apk add helm"
                sh "apk add git"
                sh "apk add curl"
                sh "apk add openjdk21"
            }
        }

        stage('Set variables and credentials') {
            steps {
                sh "git config --global --add safe.directory \$(pwd)"
                script {
                    TAG_FINAL = "${generateTag()}"
                }
                sh "echo $KUBE_CONFIG > /home/jenkins/agent/kubeconfig_location"
                sh "mkdir /home/jenkins/agent/.kube"
                sh "cp \$(cat /home/jenkins/agent/kubeconfig_location) /home/jenkins/agent/.kube/config"
            }
        }

        stage("Build Image") {
            steps {
                container('kaniko') {
                    sh "/kaniko/executor --dockerfile Dockerfile --context `pwd` --destination registry.jorgesanchezcarlin.es:443/mcs-manager-daemon:$TAG_FINAL"
                }
            }
        }

        stage('Description') {
            steps {
                script {
                    currentBuild.description = "Okey"
                }
            }
        }
    }
}

String generateTag() {
    return "${getCommitHashShort()}.${date()}"
}

String getCommitHashShort() {
    return sh (script: "git rev-parse --short HEAD", returnStdout: true).trim()
}

String date() {
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyMMddHHmm")
    return simpleDateFormat.format(new Date())
}
