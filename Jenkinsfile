pipeline {
  agent none
  stages {
    stage('Docker Build') {
      matrix {
        agent any
        axes {
          axis {
            name 'PLATFORM'
            values 'windowsservercore-ltsc2019', 'windowsservercore-1909'
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
                environment {
                    DOCKER_FILE = ".\\${JDK_VERSION}\\${TYPE}\\windows\\${PLATFORM}\\Dockerfile.${JDK_TYPE}.releases.full"
                    FULL_JDK_VERSION = getJavaVersion(env.DOCKER_FILE)
                    TAG_STRING = "-t ${getTags(JDK_VERSION, FULL_JDK_VERSION, TYPE, JDK_TYPE, PLATFORM).join(' -t ')}"
                }
                steps { 
                  script {
                    publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build', status: "IN_PROGRESS"
                    echo "Do Build for ${PLATFORM} / ${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}"
                    echo env.TAG_STRING
                  
                    bat "docker build -f ${env.DOCKER_FILE} ${env.TAG_STRING} c:\\temp\\"
                    infra.withDockerCredentials {
                      getTags(JDK_VERSION, env.FULL_JDK_VERSION, TYPE, JDK_TYPE, PLATFORM).each{ tag -> 
                        bat "docker push ${tag}"
                      }
                    }
                    publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build', status: "COMPLETED"
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

def getTags(jdkShortVersion, jdkLongVersion, type, jdkType, platform) {
  def tags = []
  if (BRANCH_NAME == 'master') {
    tags << "jenkins4eval/openjdk:${jdkShortVersion}-${type}-${jdkType}-${platform}"  
    tags << "jenkins4eval/openjdk:${jdkLongVersion}-${type}-${jdkType}-${platform}"
    if (jdkShortVersion == '15') {
      tags << "jenkins4eval/openjdk:${type}-${jdkType}-${platform}"
      if (jdkType == 'hotspot') {
        tags << "jenkins4eval/openjdk:${type}-${platform}"
        if (type == 'jdk') {
          tags << "jenkins4eval/openjdk:${platform}"
        }
      }  
    }
    if (type == 'jdk') {
      tags << "jenkins4eval/openjdk:${jdkShortVersion}-${jdkType}-${platform}"
      tags << "jenkins4eval/openjdk:${jdkLongVersion}-${jdkType}-${platform}"
    }
  } else {
    tags << "jenkins4eval/openjdk:${jdkShortVersion}-${type}-${jdkType}-SNAPSHOT"
  }
  return tags
}
