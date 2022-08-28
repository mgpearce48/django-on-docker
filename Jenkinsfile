pipeline {
    agent any
    environment {
        CI = 'true'
        DOCKERHUB_CREDENTIALS = credentials('mgpearce-dockerhub')
    }
    stages {
        stage('Build & Test') {
              steps {
                  echo 'Create images & containers for testing...'
                  sh 'docker-compose -f docker-compose.prod.yml up -d --build'
                  sh 'docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput'
                  sh 'docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --no-input --clear'
                  input message: 'Finished reviewing the react app? (Click "Proceed" to continue)'
                  sh 'docker-compose -f docker-compose.prod.yml down -v'
              }
        }
        stage('Login Dockerhub') {
            steps {
                echo 'Login to Dockerhub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push Dockerhub') {
            steps {
                echo 'Push images to Dockerhub...'
                sh 'docker tag django-on-docker-web mgpearce/django-on-docker-web'
                sh 'docker tag django-on-docker-nginx mgpearce/django-on-docker-nginx'
                sh 'docker tag postgres:14.0-alpine mgpearce/django-on-docker-db'
                sh 'docker push mgpearce/django-on-docker-web'
                sh 'docker push mgpearce/django-on-docker-nginx'
                sh 'docker push mgpearce/django-on-docker-db'
            }
        }
    }
    post {
        always {
            echo 'Logout of Dockerhub...'
            sh 'docker logout'
            cleanWs(notFailBuild: true)
        }
    }
}
