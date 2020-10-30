pipeline {
    agent none
    stages {
        stage('Docker Build') {
            parallel {
                stage('windows-2019 8') {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    steps {
                        dockerBuild(8)
                    }
                }
                stage('windows-2019 11') {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    steps {
                        dockerBuild(11)
                    }
                }
                stage('windows-2019 14') {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    steps {
                        dockerBuild(14)
                    }
                }
                stage('windows-2019 15') {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    steps {
                        dockerBuild(15)
                    }
                }
            }
        }
        stage('Docker Manifest') {
            parallel {
                stage("Manifest 8") {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    environment {
                    DOCKER_CLI_EXPERIMENTAL = "enabled"
                    }
                    steps {
                        dockerManifest(8)
                    }
                }
                stage("Manifest 11") {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    environment {
                    DOCKER_CLI_EXPERIMENTAL = "enabled"
                    }
                    steps {
                        dockerManifest(11)
                    }
                }
                stage("Manifest 14") {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    environment {
                    DOCKER_CLI_EXPERIMENTAL = "enabled"
                    }
                    steps {
                        dockerManifest(14)
                    }
                }
                stage("Manifest 15") {
                    agent {
                        label "amd64&&windock&&windows"
                    }
                    environment {
                    DOCKER_CLI_EXPERIMENTAL = "enabled"
                    }
                    steps {
                        dockerManifest(15)
                    }
                }
            }
        }
    }
}

def dockerBuild(version) {
    // dockerhub is the ID of the credentials stored in Jenkins
    docker.withRegistry('https://index.docker.io/v1/', 'jenkins-dockerhub') {
        git poll: false, url: 'https://github.com/jenkins-infra/openjdk-docker.git'
        if (version){
            sh label: '', script: "./build_all.sh ${version}"
        } else {
            sh label: '', script: "./build_all.sh"
        }
    }
}

def dockerManifest(version) {
    // dockerhub is the ID of the credentials stored in Jenkins
    docker.withRegistry('https://index.docker.io/v1/', 'jenkins-dockerhub') {
        git poll: false, url: 'https://github.com/jenkins-infra/openjdk-docker.git'
        sh label: '', script: "./update_manifest_all.sh ${version}"
    }
}
