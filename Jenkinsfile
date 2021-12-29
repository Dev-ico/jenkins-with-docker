pipeline {
    agent { label 'master' }
    environment {
	    DOCKERHUB_CREDENTIALS=credentials('docker_hub_login')
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
		        sh 'docker build -t devaico/train-schedule:${env.BUILD_NUMBER} .'
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
		        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push devaico/train-schedule:${env.BUILD_NUMBER}'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh 'echo "docker pull devaico/train-schedule:${env.BUILD_NUMBER}" | sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$staging'
                        try {
                            sh 'sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$staging "docker stop train-schedule"'
                            sh 'sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$staging "docker rm train-schedule"'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh 'sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$staging "docker run --restart always --name train-schedule -p 8080:8080 -d devaico/train-schedule:${env.BUILD_NUMBER}"'
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
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker pull devaico/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$production \"docker run --restart always --name train-schedule -p 8080:8080 -d devaico/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
