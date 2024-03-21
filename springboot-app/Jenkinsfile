pipeline {
  agent {
    docker {
      image 'adeanand2552/docker-maven-image:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/adeanand2552/cicd-projects.git'
      }
    }
    stage('Build and Test') {
      steps {
        // build the project and create a JAR file
        sh 'cd springboot-app && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://<IP>:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarQube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd springboot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "adeanand2552/springboot-app:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "springboot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerHUB')
      }
      steps {
        script {
            sh 'cd springboot-app && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerHUB") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "cicd-projects"
            GIT_USER_NAME = "adeanand2552"
        }
        steps {
            withCredentials([string(credentialsId: 'gitHUB', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "adeanand2255@gmail.com"
                    git config user.name "Ade Anand"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" sprintboot-app/springboot-app-manifests/deployment.yml
                    git add sprintboot-app/springboot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
