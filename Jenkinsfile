pipeline {
  agent any
  tools {
    jdk 'JDK17'
  }
  environment {
    sonarScanner = tool 'sonar-scanner' 
    APP_NAME = "jenkins-demo"
    RELEASE = "1.0.0"
    DOCKER_USER = "amisha908"
    DOCKER_PASS = "docker-cred"
    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}" 
    IMAGE_TAG = "${RELEASE}" + "-" + "${BUILD_NUMBER}"
  }
  triggers {
    // Trigger the pipeline on every commit to the master branch
    githubPush()
  }
  stages {
    stage('Git Checkout') {
      steps {
        git 'https://github.com/amisha908/WebAppDocker.git'  
      }
    }
    
      stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv(installationName: 'sonar') {
          sh """${sonarScanner}/bin/sonar-scanner \
            -Dsonar.projectKey=DotnetApp \
            -Dsonar.projectName=DotnetApp"""
        }
      }
     }
    stage('Build & Push Docker Image') {
      steps {
        script{
            withDockerRegistry(credentialsId: 'docker-cred') {
                docker_image = docker.build "${IMAGE_NAME}"
            }
             withDockerRegistry(credentialsId: 'docker-cred') {
                docker_image.push("${IMAGE_TAG}")
                docker_image.push('latest')
            }
        }
      }
    }

      stage('Run Docker Container'){
        steps{
            script{
                  // Stop and remove the existing container if it exists
                  sh 'docker rm -f jenkinsPipeLineDemo || true'
                  docker.image("${IMAGE_NAME}:${IMAGE_TAG}").pull()
                  //Got the error here - Administrators can decide whether to approve or reject this signature.
                  // def myContainer = docker.container('jenkinsPipeLineDemo').withRun('-p 1000:80') 
            }
        }
        post {
            always {
              // Run the Docker container with port mapping
              sh 'docker run -d -p 1000:80 --name jenkinsPipeLineDemo ${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }
    }
    
  }
}
