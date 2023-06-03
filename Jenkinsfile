pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('gitlab-credentials')
        VERSION = "1.0.0"
        CI_REGISTRY_IMAGE = "registry.gitlab.com/rubenalbi/node-todo-cicd"
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
            }
        }
    }
}
