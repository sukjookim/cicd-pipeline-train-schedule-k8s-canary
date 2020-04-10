pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "192.168.0.106:32002/nexus-docker-repo/website/d20200410/train"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("$DOCKER_IMAGE_NAME:$BUILD_NUMBER")
                    app.inside {
                        sh 'hostname'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('http://192.168.0.106:32002', 'nexus_docker') {
                        app.push()
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yaml',
                    enableConfigSubstitution: true
                    
                )
            }
        }
    }
}
