pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('gitlab-credentials')
        VERSION = "1.0.0"
        CI_REGISTRY_IMAGE = "registry.gitlab.com/rubenalbi/node-todo-cicd"
        CONTAINER_NAME = "node-todo-cicd"
        tag = "${env.BRANCH_NAME}" 
    }

    stages{
        stage('Build and Test'){
            steps {
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
                        sshCommand remote: remote, command: 'docker run --name ' + env.CONTAINER_NAME + ' -d -p 8000:8000 ' + env.CI_REGISTRY_IMAGE + ':' + env.VERSION + '-' + env.tag
                    }
                }
            }
        }
    }
}
