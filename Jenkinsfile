pipeline {
  environment {
    DOCKER_ID = "7tom3"
    DOCKER_IMAGE_1 = "cast-service"
    DOCKER_IMAGE_2 = "movie-service"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
agent any
stages {
  stage('Lancement uniquement en master') {
    when {
      branch 'master'
    }
  }
  stage('Docker Build'){
    steps {
      script {
      sh '''
        docker rm -f jenkins
        echo "BUILD $DOCKER_IMAGE_1"
        docker build -f $DOCKER_IMAGE_1/Dockerfile -t $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG $DOCKER_IMAGE_1/
      
        echo "BUILD $DOCKER_IMAGE_2"
        docker build -f $DOCKER_IMAGE_2/Dockerfile -t $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG $DOCKER_IMAGE_2/
      '''
      }
    }
  }
  stage('Docker Push'){
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    steps {
      script {
        sh '''
        docker login -u $DOCKER_ID -p $DOCKER_PASS
        docker push $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG
        docker push $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG
        '''
      }
    }
  }
  stage('Deploiement en dev'){
    environment {
      KUBECONFIG = credentials("config")
    }
    steps {
      script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        
        helm upgrade --install cast-service-dev helm/cast-service/ -n dev --create-namespace --set image.tag=$DOCKER_TAG
        helm upgrade --install movie-service-dev helm/movie-service/ -n dev --create-namespace --set image.tag=$DOCKER_TAG
        '''
      }
    }
  }
  stage('Deploiement en qa'){
    environment {
      KUBECONFIG = credentials("config")
    }
    steps {
      script {
        sh '''
        rm -Rf .kube
        mkdir .kube

        helm upgrade --install cast-service-qa helm/cast-service/ -n qa --create-namespace --set image.tag=$DOCKER_TAG
        helm upgrade --install movie-service-qa helm/movie-service/ -n qa --create-namespace --set image.tag=$DOCKER_TAG
        '''
      }
    }
  }
  stage('Deploiement en staging'){
    environment {
      KUBECONFIG = credentials("config")
    }
    steps {
      script {
        sh '''
        rm -Rf .kube
        mkdir .kube

        helm upgrade --install cast-service-staging helm/cast-service/ -n staging --create-namespace --set image.tag=$DOCKER_TAG
        helm upgrade --install movie-service-staging helm/movie-service/ -n staging --create-namespace --set image.tag=$DOCKER_TAG
        '''
      }
    }
  }
  stage('Deploiement en prod'){
    environment {
      KUBECONFIG = credentials("config")
    }
    steps {
      timeout(time: 15, unit: "MINUTES") {
          input message: 'Souhaitez-vous déployer en production ?', ok: 'Yes'
      }
      script {
        sh '''
        rm -Rf .kube
        mkdir .kube

        helm upgrade --install cast-service-prod helm/cast-service/ -n prod --create-namespace --set image.tag=$DOCKER_TAG
        helm upgrade --install movie-service-prod helm/movie-service/ -n prod --create-namespace --set image.tag=$DOCKER_TAG
        '''
      }
    }
  }
}
}
