pipeline {
    agent none
    stages {
        stage('Docker Build') {
            matrix {
                agent any
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'windows-2019'
                    }
                    axis {
                        name 'JDK_VERSION'
                        values '8', '11', '14', '15'
                    }
                    axis {
                        name 'JDK_TYPE'
                        values 'hotspot', 'openj9'
                    }
                    axis {
                        name 'TYPE'
                        values 'jdk', 'jre'
                    }
                }
                stages {
                    stage('Build') {
                        agent {
                            label "amd64&&windock&&windows"
                        }
                        steps {
                            echo "Do Build for ${PLATFORM} / ${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}"
                            bat "docker build -f .\\${JDK_VERSION}\\${TYPE}\\windows\\windowsservercore-ltsc2019\\Dockerfile.${JDK_TYPE}.releases.full -t jenkins4eval/openjdk:${JDK_VERSION}-${TYPE}-${JDK_TYPE}-windowsservercore-ltsc2019 c:\\temp\\"
                            script {
                                infra.withDockerCredentials {
                                    withEnv(['DOCKERHUB_ORGANISATION=jenkins4eval','DOCKERHUB_REPO=openjdk']) {
                                        bat "docker push jenkins4eval/openjdk:${JDK_VERSION}-${TYPE}-${JDK_TYPE}-windowsservercore-ltsc2019"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}

def dockerBuild(version) {
    infra.withDockerCredentials {
        withEnv(['DOCKERHUB_ORGANISATION=jenkins4eval','DOCKERHUB_REPO=openjdk']) {
            git poll: false, url: 'https://github.com/jenkins-infra/openjdk-docker.git'
            if (version){
                bat label: '', script: "build_all.sh ${version}"
            } else {
                bat label: '', script: "build_all.sh"
            }
        }
    }
}

def dockerManifest(version) {
    // dockerhub is the ID of the credentials stored in Jenkins
    infra.withDockerCredentials {
        withEnv(['DOCKERHUB_ORGANISATION=jenkins4eval','DOCKERHUB_REPO=openjdk']) {
            git poll: false, url: 'https://github.com/jenkins-infra/openjdk-docker.git'
            bat label: '', script: "update_manifest_all.sh ${version}"
        }
    }
}
