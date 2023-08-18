def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    environment {
        WORKSPACE = "${env.WORKSPACE}"
    }

    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/Sebastine-Atemnkeng/full-devops-cicd-pipeline-tomcat.git'

            }
        }

        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn -U clean package'
            }

            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Checkstyle Code Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('SonarQube scanning') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=maven \
                    -Dsonar.host.url=http://10.0.0.158:9000 \
                    -Dsonar.login=$SONAR_TOKEN
                    """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Upload artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                sh "sed -i \"s/.*<username><\\/username>/<username>$USER_NAME<\\/username>/g\" ${WORKSPACE}/nexus-setup/settings.xml"
                sh "sed -i \"s/.*<password><\\/password>/<password>$PASSWORD<\\/password>/g\" ${WORKSPACE}/nexus-setup/settings.xml"
                sh 'cp ${WORKSPACE}/nexus-setup/settings.xml /var/lib/jenkins/.m2'
                sh 'mvn clean deploy -DskipTests'
                }
               
            }
        }

        stage('Deploy to DEV env') {
            environment {
                HOSTS = 'dev'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ansible-deploy-server-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    sh "ansible dev_host -m copy -a 'src=/var/lib/jenkins/workspace/app-cicd-pipeline/webapp/target/webapp.war dest=/var/lib/tomcat/webapps owner=ansadmin group=ansadmin mode=0644'"
                }
            }
        }

        stage('Deploy to STAGE env') {
            environment {
                HOSTS = 'stage'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ansible-deploy-server-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    sh "ansible stage_host -m copy -a 'src=/var/lib/jenkins/workspace/app-cicd-pipeline/webapp/target/webapp.war dest=/var/lib/tomcat/webapps owner=ansadmin group=ansadmin mode=0644'"
                }
            }
        }

        stage('Approval') {
            steps {
                input('Do you want to proceed?')
            }
        }

        stage('Deploy to PROD env') {
            environment {
                HOSTS = 'prod'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ansible-deploy-server-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    sh "ansible prod_host -m copy -a 'src=/var/lib/jenkins/workspace/app-cicd-pipeline/webapp/target/webapp.war dest=/var/lib/tomcat/webapps owner=ansadmin group=ansadmin mode=0644'"
                }
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            slackSend channel: 'cybergoat', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}