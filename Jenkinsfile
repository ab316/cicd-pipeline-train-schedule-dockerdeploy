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
        
        stage('Docker Build') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build('abaig316/train-schedule')
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        
        stage('Docker Push') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull abaig316/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d abaig316/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
