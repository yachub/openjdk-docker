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
                    fullJdkVersion = getJavaVersion(dockerFile)
                    tags = getTags(JDK_VERSION, fullJdkVersion, TYPE, JDK_TYPE)
                    tagString = "-t ${tags.join(' -t ')}"
                  }
                  echo "Do Build for ${PLATFORM} / ${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}"
                  echo tagString
                  publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build'
                  bat "docker build -f ${dockerFile} ${tagString} c:\\temp\\"
                  script {
                    infra.withDockerCredentials {
                      tags.each{ tag -> 
                        bat "docker push ${tag}"
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
  def javaVersion = contents.split('\n').find{ it.startsWith('ENV JAVA_VERSION') }.replace('ENV JAVA_VERSION ','').replace('+','-')  
  def withoutJdkPrefix = javaVersion.replaceAll("^jdk-", "").replaceAll("^jdk", "")
  if (withoutJdkPrefix.contains("_")) {
    return withoutJdkPrefix.split('_')[0]
  }
  return withoutJdkPrefix
}

def getTags(jdkShortVersion, jdkLongVersion, type, jdkType) {
  def tags = []
  if (BRANCH_NAME == 'master') {
    tags << "jenkins4eval/openjdk:${jdkShortVersion}-${type}-${jdkType}-windowsservercore-ltsc2019"  
    tags << "jenkins4eval/openjdk:${jdkLongVersion}-${type}-${jdkType}-windowsservercore-ltsc2019"
    if (jdkShortVersion == '15') {
      tags << "jenkins4eval/openjdk:${type}-${jdkType}-windowsservercore-ltsc2019"
      if (jdkType == 'hotspot') {
        tags << "jenkins4eval/openjdk:${type}-windowsservercore-ltsc2019"
        if (type == 'jdk') {
          tags << "jenkins4eval/openjdk:windowsservercore-ltsc2019"
        }
      }  
    }
  } else {
    tags << "jenkins4eval/openjdk:${jdkShortVersion}-${type}-${jdkType}-SNAPSHOT"
  }
  return tags
}
