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

        stage('SonarQube analysis') {
            environment {
              scannerHome = tool 'sonarqube scanner' // the name you have given the Sonar Scanner (in Global Tool Configuration)
            }
            steps {
                withSonarQubeEnv(installationName: 'sonarqube server') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ci-code-report \
                   -Dsonar.projectName=ci-code-report-jenkins \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
}
