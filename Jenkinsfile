pipeline {
  agent any

  stages {

   stage('Cleanup Workspace') {
     steps {
       cleanWs deleteDirs: true
     }
   }


    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AshwiniEthiraj3/jenkins-cicd.git'
      }
    }

    stage('Build & Test') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17'
          args "-u \$(id -u):\$(id -g) -v /var/run/docker.sock:/var/run/docker.sock"
        }
      }
      steps {
        sh 'mvn clean package -Dcheckstyle.skip=true -DskipTests'
      }
    }

    stage('Sonar Scan') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17'
          args "-u \$(id -u):\$(id -g) -v /var/run/docker.sock:/var/run/docker.sock"
        }
      }
      environment {
        SONAR_URL = "http://3.80.73.56:9000/"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -DskipTests -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }

    stage('Build & Push Docker Image') {
      agent any
      environment {
        DOCKER_IMAGE = "ashwiniethiraj/ultimate-cicd1:${BUILD_NUMBER}"
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            docker build -t $DOCKER_IMAGE .
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker push $DOCKER_IMAGE
          '''
        }
      }
    }

    stage('Update Deployment File') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'TOKEN')]) {
          sh '''
            git config user.email "ashwiniethiraj@gmail.com"
            git config user.name "Ashwini Ethiraj3"
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
            git add .
            git commit -m "Update image tag ${BUILD_NUMBER}" || echo "No changes to commit"
            git push https://${TOKEN}@github.com/AshwiniEthiraj3/jenkins-cicd HEAD:main
          '''
        }
      }
    }
  }
}
  