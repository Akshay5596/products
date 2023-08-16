pipeline {
  agent {
    docker {
      image 'akshaypatil5596/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        // git tool: 'Default Git', url: 'https://github.com/Akshay5596/Jenkins-Zero-To-Hero'
        git branch: 'main', url: 'https://github.com/Akshay5596/products.git'
      }
    }

    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "akshaypatil5596/products:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
                    }
      }
    }
    // java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "products"
            GIT_USER_NAME = "Akshay5596"
        }
        steps {
            withCredentials([string(credentialsId: 'github-secret', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "akshay.xyz@gmail.com"
                    git config user.name "akshay patil"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s/deployment.yml
                    git add k8s/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}