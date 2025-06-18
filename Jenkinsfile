pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout'){
           steps {
                git branch: 'sk-dev', changelog: false, poll: false, url: 'https://github.com/tecfeed/Python-CI-CD.git'
           }
        }

        stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "saurabh6870/python-todo-app:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerhub-token')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub-token") {
                dockerImage.push()
            }
        }
      }
    }

        stage('Checkout K8S manifest SCM'){
            steps {
                git branch: 'main', changelog: false, credentialsId: 'github-creds', poll: false, url: 'https://github.com/tecfeed/Python-CI-CD.git'
        
                
            }
        }
          // deploy file copies to main repo
    // deploy file copies to main repo
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Python-CI-CD"
            GIT_USER_NAME = "tecfeed"
        }
        steps {
            withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "saurabh6970@gmail.com"
                    git config user.name "tecfeed"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    
                    sed -i "s/\\(image: .*:\\).*$/\\1${BUILD_NUMBER}/" deploy/deploy.yaml
                    sed -i "s/\\(image: .*:\\).*$/\\1${BUILD_NUMBER}/" deploy/pod.yaml
                    git add deploy/deploy.yaml
                    git add deploy/pod.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
}
}