pipeline {
  environment {
    DOCKER_ID = "jan986"
    DOCKER_IMAGE = "datascientestapi"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
  agent any

  stages {
    stage('DockerBuild') {
      steps {
        script {
          sh '''
          docker rm -f fastapi_test || true
          docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
          sleep 6
          '''
        }
      }
    }

    stage('DockerRun') {
      steps {
        script {
          sh '''
          docker run -d -p 80:80 --name fastapi_test $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          sleep 10
          '''
        }
      }
    }

    stage('TestAcceptance') {
      steps {
        script {
          sh '''
          curl --fail --silent --show-error http://localhost || exit 1
          '''
        }
      }
    }

    stage('DockerPush') {
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
          echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
          docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          '''
        }
      }
    }

    stage('DeploymentInDev') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          cp fastapi/values.yaml values.yml
          sed -i.bak "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace dev
          '''
        }
      }
    }

    stage('Deployment in staging') {
      environment {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
      }
      steps {
        script {
          sh '''
          cp fastapi/values.yaml values.yml
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace staging
          '''
        }
      }
    }

    stage('Deploiement en prod'){
      environment {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
      }
      steps {
      // Create an Approval Button with a timeout of 15 minutes.
      // this requires a manual validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
            input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace prod
          '''
        }
      }
    }
  }
}