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
            stages {
              stage("build") {
                steps { 
                  script {
                    dockerFile = ".\\${JDK_VERSION}\\${TYPE}\\windows\\windowsservercore-ltsc2019\\Dockerfile.${JDK_TYPE}.releases.full"
                    fullTag = getJavaVersion(dockerFile)
                  }
                  echo "Do Build for ${PLATFORM} / ${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}"
                  publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build'
                  bat "docker build -f ${dockerFile} -t jenkins4eval/openjdk:${JDK_VERSION}-${TYPE}-${JDK_TYPE}-windowsservercore-ltsc2019 -t jenkins4eval/openjdk:${fullTag}-windowsservercore-ltsc2019 c:\\temp\\"
                }
              }
              stage("push") {
                when {
                  branch 'master'
                }
                steps {
                  script {
                    infra.withDockerCredentials {
                      withEnv(['DOCKERHUB_ORGANISATION=jenkins4eval','DOCKERHUB_REPO=openjdk']) {
                        bat "docker push ${DOCKERHUB_ORGANISATION}/${DOCKERHUB_REPO}:${JDK_VERSION}-${TYPE}-${JDK_TYPE}-windowsservercore-ltsc2019"
                        bat "docker push ${DOCKERHUB_ORGANISATION}/${DOCKERHUB_REPO}:${fullTag}-windowsservercore-ltsc2019"
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
  }
}

def getJavaVersion(path) {
   def contents = readFile file: path
   return contents.split('\n').find{ it.startsWith('ENV JAVA_VERSION') }.replace('ENV JAVA_VERSION ','').replace('+','-')  
}
