def GITLAB_STATE_SUCCESS = 'success'
def GITLAB_STATE_FAILURE = 'failed'
def PYTHON_VERSION = '3.8' // TODO: could add this to dockerfile buildArgs
def ARTIFACTORY_SERVER_ID = 'ARTIFACTORY_SERVER'

// TODO: can set this to the stages Setup section and
//       dynamically set based on agent type
String PATH_WORKAROUD = '' // windows
// String PATH_WORKAROUD = '$env:PATH="/root/.local/bin:"+$env:PATH' // unix

pipeline {
    agent {
        dockerfile {
            filename 'Dockerfile.windows'
            reuseNode true
            // TODO: can move this to the stages section and
            //       dynamically set user based on agent type
            args '-u ContainerAdministrator' // windows
            // args '-u root' // unix

            // Agent label
            label '<if-you-use-agent-labels>'
        }
    }

    options {
        gitLabConnection('GitLabAPIConnection')
        gitlabBuilds(builds: ['lint', 'test', 'build'])
        timestamps()
        timeout(time: 5, unit: 'MINUTES')   // timeout on whole pipeline job
        ansiColor('xterm') // color the console output in jenkins ui
    }

    stages {
        stage('Setup') {
            steps {
                pwsh """
                $PATH_WORKAROUD
                python --version
                tox --version
                hatch --version
                """
            }
        }
        stage('Lint') {
            steps {
                pwsh """
                $PATH_WORKAROUD
                tox -e lint
                """
            }
            post {
                failure {
                    updateGitlabCommitStatus name: 'lint', state: "$GITLAB_STATE_FAILURE"
                }
                success {
                    updateGitlabCommitStatus name: 'lint', state: "$GITLAB_STATE_SUCCESS"
                }
            }
        }
        stage('Test') {
            steps {
                pwsh """
                $PATH_WORKAROUD
                tox
                """
            }
            post {
                failure {
                    updateGitlabCommitStatus name: 'test', state: "$GITLAB_STATE_FAILURE"
                }
                success {
                    updateGitlabCommitStatus name: 'test', state: "$GITLAB_STATE_SUCCESS"
                }
            }
        }
        stage('Build') {
            steps {
                pwsh """
                $PATH_WORKAROUD
                hatch build
                """
            }
            post {
                failure {
                    updateGitlabCommitStatus name: 'build', state: "$GITLAB_STATE_FAILURE"
                }
                success {
                    updateGitlabCommitStatus name: 'build', state: "$GITLAB_STATE_SUCCESS"
                }
            }
        }
        stage('Publish') {
            when { tag 'v*' }
            steps {
                rtServer(
                    id: "$ARTIFACTORY_SERVER_ID",
                    url: "https://${ArtifactoryServerUrl}/artifactory",
                    credentialsId: 'ArtifactoryCreds'
                )
                script {
                    pwsh "$PATH_WORKAROUD; hatch version >> ver.txt"
                    def PACKAGE_VERSION = pwsh(
                        script:  'Select-String -Path "ver.txt" -Pattern "\\d.*" -Raw' , // "\d.*""
                        returnStdout: true
                    ).replaceAll('\\s', '') // strip the rogue newline
                    echo "Package version is: '$PACKAGE_VERSION'"
                    rtUpload(
                        serverId: "$ARTIFACTORY_SERVER_ID",
                        spec: """{
                            "files": [
                                {
                                    "pattern": "dist/",
                                    "target": "<the-pypi>/$PACKAGE_VERSION/"
                                }
                            ]
                        }"""
                    )
                }
                rtPublishBuildInfo (
                    serverId: "$ARTIFACTORY_SERVER_ID"
                )
            }
            post {
                failure {
                    updateGitlabCommitStatus name: 'publish', state: "$GITLAB_STATE_FAILURE"
                }
                success {
                    updateGitlabCommitStatus name: 'publish', state: "$GITLAB_STATE_SUCCESS"
                }
            }
        }
    }

    post {
        always {
            jiraSendBuildInfo()
            junit 'build/test-*.xml'
        }
        changed {
            emailext body: '$PROJECT_NAME - Build $BUILD_DISPLAY_NAME: $BUILD_STATUS<br><br>Check console output at $BUILD_URL to view the results.',
                recipientProviders: [culprits(), requestor()],
                subject: '$PROJECT_NAME - Build $BUILD_DISPLAY_NAME: $BUILD_STATUS'
        }
        cleanup {
            cleanWs()
        }
    }
}
