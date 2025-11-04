pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')  // Les credentials DockerHub
        MOVIE_REPO = 'dockerformation123/movie-service'  // Repo pour movie
        CAST_REPO = 'dockerformation123/cast_service'   // Repo pour cast
        IMAGE_TAG = "${BUILD_NUMBER}"  // Tag dynamique avec numéro de build
        KUBE_CONFIG = credentials('config')  // Les credentials Kube
    }
    parameters {
        choice(name: 'ENV', choices: ['dev', 'qa', 'staging', 'prod'], description: 'Environnement de déploiement')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Push Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        // Build et push pour movie-service
                        sh "docker build -t ${MOVIE_REPO}:${IMAGE_TAG} movie-service/"
                        sh "docker push ${MOVIE_REPO}:${IMAGE_TAG}"
                        
                        // Build et push pour cast-service
                        sh "docker build -t ${CAST_REPO}:${IMAGE_TAG} cast-service/"
                        sh "docker push ${CAST_REPO}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Deploy Non-Prod') {
            when { not { branch 'master' } }  // Auto pour non-master
            steps {
                sh "echo '${KUBE_CONFIG}' > kubeconfig && chmod 600 kubeconfig"
                sh "helm upgrade --install app ./charts --namespace ${params.ENV} --set movie.image.repository=${MOVIE_REPO} --set movie.image.tag=${IMAGE_TAG} --set cast.image.repository=${CAST_REPO} --set cast.image.tag=${IMAGE_TAG} --kubeconfig kubeconfig"
            }
        }
        stage('Approve Prod') {
            when { branch 'master' }  // Manuel pour prod
            steps {
                input message: 'Approuver le déploiement en Prod ?', ok: 'Oui'
            }
        }
        stage('Deploy Prod') {
            when { branch 'master' }
            steps {
                sh "echo '${KUBE_CONFIG}' > kubeconfig && chmod 600 kubeconfig"
                sh "helm upgrade --install app ./charts --namespace prod --set movie.image.repository=${MOVIE_REPO} --set movie.image.tag=${IMAGE_TAG} --set cast.image.repository=${CAST_REPO} --set cast.image.tag=${IMAGE_TAG} --kubeconfig kubeconfig"
            }
        }
    }
}