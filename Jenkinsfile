def COLOR_MAP = [
<<<<<<< HEAD
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
                git branch: 'main', url: 'https://github.com/cvamsikrishna11/devops-fully-automated.git'

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
                    -Dsonar.host.url=http://172.31.20.13:9000 \
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
                    sh "ansible-playbook -i ${WORKSPACE}/ansible-setup/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
                }
            }
        }

        stage('Deploy to STAGE env') {
            environment {
                HOSTS = 'stage'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'ansible-deploy-server-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    sh "ansible-playbook -i ${WORKSPACE}/ansible-setup/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
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
                    sh "ansible-playbook -i ${WORKSPACE}/ansible-setup/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
                }
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            slackSend channel: '#team-devops', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
=======
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
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
      post {
        success {
          echo ' now Archiving '
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }
    stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    stage('SonarQube Scan') {
      steps {
        sh """mvn sonar:sonar \
              -Dsonar.projectKey=JavaWebApp \
              -Dsonar.host.url=http://172.31.4.143:9000 \
              -Dsonar.login=e9733df3fcd6ed54cef307d8ac4cc00eeb2d3611"""
      }
    }
    stage('Upload to Artifactory') {
      steps {
        sh "mvn clean deploy -DskipTests"
      }
    }
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
    // stage('Approval for stage') {
    //   steps {
    //     input('Do you want to proceed?')
    //   }
    // }
    stage('Deploy to Stage') {
      environment {
        HOSTS = "stage" // Make sure to update to "stage"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
    stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to PROD') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
  }
  post {
    always {
        echo 'Slack Notifications.'
        slackSend channel: '#mbandi-jenkins-cicd-pipeline-alerts', //update and provide your channel name
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
    }
  }
}

//slackSend channel: '#mbandi-cloudformation-cicd', message: "Please find the pipeline status of the following ${env.JOB_NAME ${env.BUILD_NUMBER} ${env.BUILD_URL}"
>>>>>>> dd1ad615b148f7c3fa85533d401a2f65a2e2715a
