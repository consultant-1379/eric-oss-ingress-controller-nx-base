#!/usr/bin/env groovy

def defaultBobImage = 'armdocker.rnd.ericsson.se/proj-adp-cicd-drop/bob.2.0:1.7.0-52'
def bob = new BobCommand()
        .bobImage(defaultBobImage)
        .envVars([HELM_REPO_TOKEN:'${HELM_REPO_TOKEN}',
                  RELEASE:'${RELEASE}',
                  GERRIT_CHANGE_NUMBER:'${GERRIT_CHANGE_NUMBER}',
                  KUBECONFIG:'${KUBECONFIG}',
                  USER:'${USER}'])
        .needDockerSocket(true)
        .toString()

pipeline {
    agent {
            label 'Cloud-Native'
        }

    stages {
        stage('Clean') {
                steps {
                       sh "${bob} clean"
              }
        }
        stage('Init') {
            steps {
                sh "${bob} init-review"
            }
        }
        stage('Build') {
            steps {
              sh "${bob} build"
            }
        }
        stage('Generate') {
            steps {
                echo "Generate docs needs to be implemented"
            }
        }
        stage('Image') {
            steps {             
                sh "${bob} image"
            }
        }
        stage('Test') {
            steps {
                  sh "${bob} test"
            }
        }
    }
    post {
        always {
            script {
               sh "${bob} remove"
               archiveArtifacts 'artifact.properties'
               archive "build/image-dr/image-design-rule-check-report.*"
             }
        }
        failure {
            mail to: 'Galaxy@tcscomprod.onmicrosoft.com',
                    subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                    body: "Failure on ${env.BUILD_URL}"
        }
    }
}

// More about @Builder: http://mrhaki.blogspot.com/2014/05/groovy-goodness-use-builder-ast.html
import groovy.transform.builder.Builder
import groovy.transform.builder.SimpleStrategy

@Builder(builderStrategy = SimpleStrategy, prefix = '')
class BobCommand {
    def bobImage = 'bob.2.0:latest'
    def envVars = [:]
    def needDockerSocket = false

    String toString() {
        def env = envVars
                .collect({ entry -> "-e ${entry.key}=\"${entry.value}\"" })
                .join(' ')

        def cmd = """\
            |docker run
            |--init
            |--rm
            |--workdir \${PWD}
            |--user \$(id -u):\$(id -g)
            |-v \${PWD}:\${PWD}
            |-v /etc/group:/etc/group:ro
            |-v /etc/passwd:/etc/passwd:ro
            |-v \${HOME}/.m2:\${HOME}/.m2
            |-v \${HOME}/.docker:\${HOME}/.docker
            |${needDockerSocket ? '-v /var/run/docker.sock:/var/run/docker.sock' : ''}
            |${env}
            |\$(for group in \$(id -G); do printf ' --group-add %s' "\$group"; done)
            |--group-add \$(stat -c '%g' /var/run/docker.sock)
            |${bobImage}
            |"""
        return cmd
                .stripMargin()           // remove indentation
                .replace('\n', ' ')      // join lines
                .replaceAll(/[ ]+/, ' ') // replace multiple spaces by one
    }
}
  
