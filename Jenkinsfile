def COLOR_MAP = [
    'SUCCESS' : 'good',
    'FAILURE' : 'danger',
]

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
                sh "mvn checkstyle:checkstyle"
            }
        }

        stage('SonarQube analysis') {
            environment {
              scannerHome = tool 'sonarqube scanner' // the name you have given the Sonar Scanner (in Global Tool Configuration)
            }
            steps { // the name you have given the Sonar Server (in configure system)
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

        stage('Nexus Upload Artifact') {
            steps{
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: "http",
                nexusUrl: '172.31.4.237:8081',
                groupId: 'artifact upload', // under groupId folder comes artifactId folder
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: 'artifact-repo', // maven2(hosted) repo name
                credentialsId: 'nexuslogin',
                artifacts: [
                  [artifactId: "ci-jenkins",
                  classifier: '',
                  file: 'target/vprofile-v2.war', // artifact path in jenkins
                  type: 'war']
                ]
               )
            }
        }
    }

    post {
      always {
        echo "slack notification for ci-jenkins"
        slackSend channel: "#ci-jenkins"
           color: COLOR_MAP[currentBuild.currentResult],
           message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More Info At: ${env.BUILD_URL}"
      }
    }
}
