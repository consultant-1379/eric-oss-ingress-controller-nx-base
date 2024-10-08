#!/usr/bin/env groovy

def defaultBobImage = 'armdocker.rnd.ericsson.se/proj-adp-cicd-drop/bob.2.0:1.7.0-52'
def bob = new BobCommand()
        .bobImage(defaultBobImage)
        .envVars([ISO_VERSION: '${ISO_VERSION}',
                  IMAGE_TAG:'${IMAGE_TAG}',
                  IMAGE_REPO:'${IMAGE_REPO}',
                  IMAGE_NAME:'${IMAGE_NAME}',
                  GERRIT_USERNAME:'${GERRIT_USERNAME}',
                  GERRIT_PASSWORD:'${GERRIT_PASSWORD}',
                  GERRIT_CHANGE_NUMBER:'${GERRIT_CHANGE_NUMBER}'])
        .needDockerSocket(true)
        .toString()

pipeline {
    agent {
        node {
            label "Cloud_Native_Dev"
        }
    }

    parameters {
        string(name: 'IMAGE_TAG', description: 'The image tag for base OS (e.g. 1.0.0-7)')
    }

    stages {
        stage('Update Base OS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'cenmbuild_gerrit_api_token',
                                 usernameVariable: 'GERRIT_USERNAME',
                                 passwordVariable: 'GERRIT_PASSWORD')])
                {
                    sh "${bob} create-new-cbo-patch"
                }
            }
        }
    }
}
// More about @Builder: http://mrhaki.blogspot.com/2014/05/groovy-goodness-use-builder-ast.html
import groovy.transform.builder.Builder
import groovy.transform.builder.SimpleStrategy

@Builder(builderStrategy = SimpleStrategy, prefix = '')
class BobCommand {

    def bobImage = 'bob.2.0:1.7.0-52'
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
            |${bobImage}
            |"""
        return cmd
                .stripMargin()           // remove indentation
                .replace('\n', ' ')      // join lines
                .replaceAll(/[ ]+/, ' ') // replace multiple spaces by one
    }
}