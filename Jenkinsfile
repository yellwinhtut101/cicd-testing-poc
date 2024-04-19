pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    // AN_ACCESS_KEY = credentials('ywh-credentials')
    AWS_DEFAULT_REGION    = 'ap-southeast-1'
    IMAGE_NAME            = 'yellwinhtut/jenkins-example-laravel'
    IMAGE_TAG             = 'v1.0'
    ECR_REPO              = '006961800653.dkr.ecr.us-east-1.amazonaws.com/testing-repo'
    EC2_INSTANCE_IP       = '54.80.70.193'
    // SSH_CREDENTIALS       = credentials('your-ssh-credentials')
    GIT_COMMIT_SHORT = sh(script: "printf \$(git rev-parse --short ${GIT_COMMIT})", returnStdout: true).trim()
  }

  // tools {
  //       nodejs 'node' // Assuming you have Node.js tool configured in Jenkins
  // }

  stages {
    stage('Build') {
      steps {
        sh 'sudo -i'
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }
    stage('Test') {
      steps {
        sh 'echo "Testing 1 2 3"'
      }
    }
    stage('SonarQube analysis') {
      environment {
          SCANNER_HOME = tool 'Sonar-Jenkins'
      }
      steps {
        withSonarQubeEnv(credentialsId: 'Sonar-Jenkins', installationName: 'Sonar') {
            sh '''$SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=projectKey \
                -Dsonar.projectName=projectName \
                -Dsonar.sources=src/ \
                -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Push to Amazon ECR') {
      steps {
          sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"""
          sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
          sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}