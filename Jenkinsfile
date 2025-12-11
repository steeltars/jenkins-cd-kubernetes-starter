pipeline {
    agent any
    environment {
        dockerCreds = credentials('dockerhub_login')
        registry = "${dockerCreds_USR}/vatcal"
        registryCredentials = "dockerhub_login"
        dockerImage = ""
        KUBECONFIG = "config.yaml" // added this line
    }
    stages {
        stage('Run Tests') {
            steps {
               sh 'npm install'
               sh 'npm test'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build(registry)
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("", registryCredentials) {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Clean Up') {
            steps {
                sh "docker image prune --all --force --filter 'until=48h'"
            }
        }
        stage('Deploy to Kubernetes') { // added this stage
            steps {
                sh "cp -u /mnt/k3s/config config.yaml"
                sh "kubectl apply -f kubernetes/deploy.yml"
                sh "kubectl apply -f kubernetes/service.yml"
            }
        }
    }
}
