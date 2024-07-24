pipeline {
   agent any 
   
  stages {
    stage('Clean Workspace') {
      steps {
        sshagent(credentials: [GIT_HUB_CREDENTIALS]) {
            sh "git clone --recursive git@github.com:bertiniawara/jenkins-CICD.git"
          }
        echo "clone_project"
       }
     }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'cd springbot-demo/springbot-demo/springboot-crud-app && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://10.0.0.8:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'SONAR_AUTH_TOKEN', variable: 'Password')]) {
          sh 'cd springbot-demo/springbot-demo/springboot-crud-app && mvn sonar:sonar -Dsonar.login=$Password -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "betiniawara842/ultimate-cicd:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('DOCKER_HUB_PASSWORD')
      }
      steps {
        script {
            sh 'cd springbot-demo/springbot-demo/springboot-crud-app && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "DOCKER_HUB_PASSWORD") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "jenkins-CICD"
            GIT_USER_NAME = "bertiniawara"
        }
        steps {
            withCredentials([string(credentialsId: 'GIT_HUB_CREDENTIALS', variable: 'Password')]) {
                sh '''
                    git config user.email "betiniawara@gmail.com"
                    git config user.name "bertiniawara"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${Password}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
