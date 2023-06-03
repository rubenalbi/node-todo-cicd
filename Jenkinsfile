pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('gitlab-credentials')
        CI_REGISTRY_IMAGE = "registry.gitlab.com/rubenalbi/node-todo-cicd"
        CONTAINER_NAME = "node-todo-cicd"
        tag = "${env.BRANCH_NAME}"
        DEPLOYMENT_PORT = "1000:8000"
    }

    stages{
        stage('Setup parameters') {
            steps {
                script { 
                    properties([
                        parameters([
                            choice(
                                choices: ['ONE', 'TWO'], 
                                name: 'PARAMETER_01'
                            ),
                            booleanParam(
                                defaultValue: true, 
                                description: '', 
                                name: 'BOOLEAN'
                            ),
                            text(
                                defaultValue: '''
                                this is a multi-line 
                                string parameter example
                                ''', 
                                 name: 'MULTI-LINE-STRING'
                            ),
                            string(
                                defaultValue: 'scriptcrunch', 
                                name: 'STRING-PARAMETER', 
                                trim: true
                            )
                        ])
                    ])
                }
            }
        }
        stage('Test'){
            agent {
                docker {
                    image 'node:18.16.0-alpine'
                    // Run the container on the node specified at the
                    // top-level of the Pipeline, in the same workspace,
                    // rather than on a new node entirely:
                    reuseNode true
                }
            }
            steps {
                sh 'npm --version'
                sh 'npm run test'
                sh 'apk update && apk add jq'
                script {
                    env.VERSION = sh(script: 'jq -r \".version\" < ./package.json | xargs', returnStdout: true)
                }
            }
        }
        stage('Build'){
            steps {
                echo "${params.PARAMETER_01}"
                sh 'docker build --pull -t $CI_REGISTRY_IMAGE:$VERSION-$tag .'
            }
        }
        stage('Push Image'){
            steps {
                echo 'logging in to docker hub and pushing image..'
                withCredentials([usernamePassword(credentialsId:'gitlab-credentials',passwordVariable:'dockerHubPassword', usernameVariable:'dockerHubUser')]) {
                    sh "docker login registry.gitlab.com -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push $CI_REGISTRY_IMAGE:$VERSION-$tag'
                }
            }
        }
        stage('Deploy'){
            steps {
                echo 'Deploying application'
                script {
                    echo env.CI_REGISTRY_IMAGE
                    echo params.PARAMETER_01
                    def remote = [:]
                    remote.name = 'ruben'
                    remote.host = 'homeserver.rubenalbiach.com'
                    remote.allowAnyHosts = true
                    withCredentials([usernamePassword(
                        credentialsId: 'homeserver-ssh', passwordVariable: 'userPassword', usernameVariable: 'userName')]) {
                        remote.user = userName
                        remote.password = userPassword
                        sshCommand remote: remote, command: 'docker stop ' + env.CONTAINER_NAME + ' || true'
                        sshCommand remote: remote, command: 'docker rm ' + env.CONTAINER_NAME + ' || true'
                        sshCommand remote: remote, command: 'docker image prune -f'
                        withCredentials([usernamePassword(credentialsId:'gitlab-credentials',passwordVariable:'dockerHubPassword', usernameVariable:'dockerHubUser')]) {
                            sshCommand remote: remote, command: 'docker login registry.gitlab.com -u ' + env.dockerHubUser + ' -p ' + env.dockerHubPassword
                            sshCommand remote: remote, command: 'docker pull ' + env.CI_REGISTRY_IMAGE + ':' + env.VERSION + '-' + env.tag
                            sshCommand remote: remote, command: 'docker run --name ' + env.CONTAINER_NAME + ' -d -p ' + env.DEPLOYMENT_PORT + ' ' + env.CI_REGISTRY_IMAGE + ':' + env.VERSION + '-' + env.tag
                            sshCommand remote: remote, command: 'docker logout registry.gitlab.com'
                        }
                    }
                }
            }
        }
    }
}
