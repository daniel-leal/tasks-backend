pipeline {
  agent any
  stages {
    stage('Build backend') {
      steps {
        sh 'mvn clean package -DskipTests=true'
      }
    }
    stage('Unit Tests') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Sonar Analysis') {
      environment {
        scannerHome = tool 'SONAR_SCANNER'
      }
      steps {
        withSonarQubeEnv('SONAR_LOCAL') {
          sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9001 -Dsonar.login=8939e502010f33162ccb719180034887b3d2839c -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
        }
      }
    }
    stage('Quality Gate') {
      steps {
        sleep(5)
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy Backend') {
      steps {
        deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
      }
    }
    stage('API Test') {
      steps {
        dir('api-test') {
          git credentialsId: 'GithubLogin', url: 'https://github.com/daniel-leal/tasks-api-test'
          sh 'mvn test'
        }
      }
    }
    stage('Deploy Frontend') {
      steps {
        dir('frontend') {
          git credentialsId: 'GithubLogin', url: 'https://github.com/daniel-leal/tasks-frontend'
          sh 'mvn clean package'
          deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
        }
      }
    }
    stage('Functional Test') {
      steps {
        dir('functional-test') {
          git credentialsId: 'GithubLogin', url: 'https://github.com/daniel-leal/tasks-funcional-test'
          sh 'mvn test'
        }
      }
    }
    stage('Deploy Prod') {
      steps {
        sh 'docker-compose build'
        sh 'docker-compose up -d'
      }
    }
    stage('Health Check') {
      steps {
        sleep(5)
        dir('functional-test') {
          sh 'mvn verify -Dskip.surefire.tests'
        }
      }
    }
  }
  post {
    always {
      junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
    }
  }
}


