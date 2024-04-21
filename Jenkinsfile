pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }

  environment {
    AN_ACCESS_KEY = credentials('ywh-token')
    AWS_DEFAULT_REGION    = 'ap-southeast-1'
    IMAGE_NAME            = 'yellwinhtut/jenkins-example-laravel'
    IMAGE_TAG             = 'v1.0'
    ECR_REPO              = '006961800653.dkr.ecr.ap-southeast-1.amazonaws.com/y3ll-lab'
    EC2_INSTANCE_IP       = '54.80.70.193'
    // SSH_CREDENTIALS       = credentials('your-ssh-credentials')
    GIT_COMMIT_SHORT = sh(script: "printf \$(git rev-parse --short ${GIT_COMMIT})", returnStdout: true).trim()
  }


  stages {
    stage('Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }
    stage('Install TypeScript') {
      steps {
        sh 'npm install typescript'
      }
    }
    stage('SonarQube analysis') {
      environment {
          SCANNER_HOME = tool 'SonarQube'
      }
      steps {
        withSonarQubeEnv(credentialsId: 'ywh-token', installationName: 'SonarQube') {
            sh '''$SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=sample-app \
                -Dsonar.projectName=sample-app \
                -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true

        // script {
        //             def qg = waitForQualityGate()
        //             def securityRating = qg.qualityGate.conditions.find { it.metric == 'security_rating' }.status
        //             print(qg)

        //             if (qg.status != "'OK'") {
        //                 error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
        //             } else if (securityRating == 'ERROR' || securityRating == 'WARN') {
        //                 error "Pipeline aborted due to failing security rating: ${securityRating}"
        //             } else {
        //                 echo "Quality Gate passed with security rating: ${securityRating}"
        //             }
        //             // def qgCondition = 'NEW_CODE_COVERAGE < 5' // Replace with your condition
        //             // def sonarqubeUrl = 'http://54.169.105.66:9000' // Replace with your server URL
        //             // def token = 'ywh-token' // Replace with your API token (requires Administer Quality Gate permission)

        //             // def response = sh(
        //             //     script: "curl -X GET -u ${token}:${token} ${sonarqubeUrl}/api/qualitygates/project_status?projectKey=${JOB_NAME} | jq -r '.conditions[0].status'",
        //             //     returnStdout: true
        //             // ).trim()

        //             // if (response != 'SUCCESS' && response != qgCondition) {
        //             //     error "SonarQube Quality Gate failed. Condition '${qgCondition}' not met."
        //             // }
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