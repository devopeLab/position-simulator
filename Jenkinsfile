pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean install'
                echo 'Build complete'
            }
        }

        stage('Image Build') {
            steps {
                echo 'Building Docker image.........'
                sh 'docker build -t houseen/obs/position-simulator:${commit_id} .'
                echo 'Build complete'
                echo 'Pushing Docker image to DockerHub.........'
                sh 'docker push houseen/obs/position-simulator:${commit_id}'
                echo 'Push complete'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'
                sh 'sed -i "s|robobobs/obs-fleetman-position-simulator:release2|houseen/obs/position-simulator:${commit_id}|" workloads.yaml'
                sh 'kubectl config set-context --current --namespace=kubernetes-admin@kubernetes'
                sh 'kubectl get all'
                sh 'kubectl apply -f .'
                sh 'kubectl get all'
                echo 'Deployment complete'
            }
        }
    }
}
