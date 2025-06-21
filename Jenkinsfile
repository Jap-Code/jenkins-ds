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
          rm -Rf .kube
          mkdir .kube
          export KUBECONFIG=$PWD/.kube/config
          cat $KUBECONFIG > $KUBECONFIG
          cp fastapi/values.yaml values.yml
          sed -i.bak "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace dev
          '''
        }
      }
