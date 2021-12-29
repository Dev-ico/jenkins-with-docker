pipeline {
    environment {
	    DOCKERHUB_CREDENTIALS=credentials('docker_hub_login')
	}
    stages {
        stage('Build Code') {
            agent { label 'master' }
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            agent { label 'master' }
            when {
                branch 'master'
            }
            steps {
		        sh "docker build -t devaico/train-schedule:${env.BUILD_NUMBER} ."
            }
        }
        stage('Push Docker Image') {
            agent { label 'master' }
            when {
                branch 'master'
            }
            steps {
		        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh "docker push devaico/train-schedule:${env.BUILD_NUMBER}"
            }
        }
        stage('DeployToStaging') {
            agent { label 'staging' }
            when {
                branch 'master'
            }
            steps {
                sh "docker pull devaico/train-schedule:${env.BUILD_NUMBER}"
                try {
                    sh "docker stop train-schedule"
                    sh "docker rm train-schedule"
                }
                catch (err) {
                    echo: 'caught error: $err'
                }
                sh "docker run --restart always --name train-schedule -p 8080:8080 -d devaico/train-schedule:${env.BUILD_NUMBER}"
            }
        }
        stage('DeployToProduction') {
            agent { label 'production' }
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh "docker pull devaico/train-schedule:${env.BUILD_NUMBER}"
                try {
                    sh "docker stop train-schedule"
                    sh "docker rm train-schedule"
                }
                catch (err) {
                            echo: 'caught error: $err'
                }
                sh "docker run --restart always --name train-schedule -p 8080:8080 -d devaico/train-schedule:${env.BUILD_NUMBER}"
            }
        }
    }
}
