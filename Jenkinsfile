pipeline {
    agent any
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
                    app = docker.build("192.168.0.112:8082/nexus-docker-repo/website/d20200410/train:$(git rev-parse HEAD | cut -c1-5)")
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
                    docker.withRegistry('http://192.168.0.112:8082', 'nexus_docker') {
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
                sshagent(credentials: ['ssh_prometheus_node']) {
                    script {
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@$prod_ip \"docker pull 192.168.0.112:8082/nexus-docker-repo/website/d20200410/train:latest\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no ubuntu@$prod_ip \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no ubuntu@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@$prod_ip \"docker run --restart always --name train-schedule -p 8090:8080 -d 192.168.0.112:8082/nexus-docker-repo/website/d20200410/train:latest\""
                    }
                }
            }
        }
    }
}
