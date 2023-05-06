pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "java1.8"
    }

    stages {
        stage('FETCH CODE') {
            steps {
                git branch: 'vp-rem', url: 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }
    
        stage('BUILD') {
            steps {
                sh 'mvn install -DskipTests'
            }
            
            post {
                success {
                    echo 'success archive artifacts'
                    archiveArtifacts artifacts:"**/*.war"
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        stage('CHECKSTYLE ANALYSIS') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}