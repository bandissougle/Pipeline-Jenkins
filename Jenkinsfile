pipeline {
  agent any
  stages {
    stage('SCM') {
      steps {
        checkout scm
      }
    } 
    stage('Compile') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args '-v /root/.m2/repository:/root/.m2/repository'
       // to use the same node and workdir defined on top-level pipeline for all docker agents
       reuseNode true
      }
     }
     steps { 
      sh "mvn clean compile -Dhttps.protocols=TLSv1.2 \
      -Dmaven.repo.local=/root/.m2/repository \
      -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=WARN \
      -Dorg.slf4j.simpleLogger.showDateTime=true \
      -Djava.awt.headless=true \
      --batch-mode --errors --fail-at-end --show-version -DinstallAtEnd=true -DdeployAtEnd=true"
     }
    } 
  }
}