pipeline {

  agent any

  environment {
    DOCKER_ROOT_REPOSITORY = "loic29290"
    DOCKER_REPOSITORY = "loic29290/jenkins_exam"
    DOCKER_TAG = "V${BUILD_ID}.0"
    IMAGE_MOVIE = "movie"
    IMAGE_CAST = "cast"
  }

  stages {
    stage('build image cast') {
      steps {
        sh '''
          docker rm -f $IMAGE_CAST
          docker build -t $DOCKER_REPOSITORY-$IMAGE_CAST:$DOCKER_TAG ./cast-service
        '''
      }
    }
    stage('build image movie') {
      steps {
        sh '''
          docker rm -f $IMAGE_MOVIE
          docker build -t $DOCKER_REPOSITORY-$IMAGE_MOVIE:$DOCKER_TAG ./movie-service
        '''
      }
    }

    stage('push image') {
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
            echo $DOCKER_PASS | docker login --username $DOCKER_ROOT_REPOSITORY --password-stdin
            docker push $DOCKER_REPOSITORY-$IMAGE_CAST:$DOCKER_TAG
            docker push $DOCKER_REPOSITORY-$IMAGE_MOVIE:$DOCKER_TAG
          '''
        }
      }
    }

    stage('deploy dev') {
      environment {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        KUBE_NAMESPACE = "dev"
      }
      steps {
        script {
          sh '''
            rm -Rf .kube
            mkdir .kube
            cat $KUBECONFIG > .kube/config
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_CAST/values.yaml
            helm upgrade --install $IMAGE_CAST chart/$IMAGE_CAST --values=chart/$IMAGE_CAST/values.yaml --namespace $KUBE_NAMESPACE
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_MOVIE/values.yaml
            helm upgrade --install $IMAGE_MOVIE chart/$IMAGE_MOVIE --values=chart/$IMAGE_MOVIE/values.yaml --namespace $KUBE_NAMESPACE
          '''
        }
      }
    }

    stage('deploy qa') {
      environment {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        KUBE_NAMESPACE = "qa"
      }
      steps {
        script {
          sh '''
            rm -Rf .kube
            mkdir .kube
            cat $KUBECONFIG > .kube/config
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_CAST/values.yaml
            helm upgrade --install $IMAGE_CAST chart/$IMAGE_CAST --values=chart/$IMAGE_CAST/values.yaml --namespace $KUBE_NAMESPACE
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_MOVIE/values.yaml
            helm upgrade --install $IMAGE_MOVIE chart/$IMAGE_MOVIE --values=chart/$IMAGE_MOVIE/values.yaml --namespace $KUBE_NAMESPACE
          '''
        }
      }
    }

    stage('deploy staging') {
      environment {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        KUBE_NAMESPACE = "staging"
      }
      steps {
        script {
          sh '''
            rm -Rf .kube
            mkdir .kube
            cat $KUBECONFIG > .kube/config
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_CAST/values.yaml
            helm upgrade --install $IMAGE_CAST chart/$IMAGE_CAST --values=chart/$IMAGE_CAST/values.yaml --namespace $KUBE_NAMESPACE
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_MOVIE/values.yaml
            helm upgrade --install $IMAGE_MOVIE chart/$IMAGE_MOVIE --values=chart/$IMAGE_MOVIE/values.yaml --namespace $KUBE_NAMESPACE
          '''
        }
      }
    }

    stage('deploy prod') {
      when {
        branch "master"
      }
      environment {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        KUBE_NAMESPACE = "prod"
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
            rm -Rf .kube
            mkdir .kube
            cat $KUBECONFIG > .kube/config
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_CAST/values.yaml
            helm upgrade --install $IMAGE_CAST chart/$IMAGE_CAST --values=chart/$IMAGE_CAST/values.yaml --namespace $KUBE_NAMESPACE
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" chart/$IMAGE_MOVIE/values.yaml
            helm upgrade --install $IMAGE_MOVIE chart/$IMAGE_MOVIE --values=chart/$IMAGE_MOVIE/values.yaml --namespace $KUBE_NAMESPACE
          '''
        }
      }
    }
  }
}
