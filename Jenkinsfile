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
    stage('Tests Unitaires'){
      when {
        anyOf { branch 'master'; branch 'develop' }
      }
      agent {
        docker {
        image 'maven:3.6.0-jdk-8-alpine'
        args '-v /root/.m2/repository:/root/.m2/repository'
        reuseNode true
        }
      }
      steps {
        sh "mvn test -Dhttps.protocols=TLSv1.2 \
        -Dmaven.repo.local=/root/.m2/repository \
        -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=WARN \
        -Dorg.slf4j.simpleLogger.showDateTime=true \
        -Djava.awt.headless=true \
        --batch-mode --errors --fail-at-end --show-version -DinstallAtEnd=true -DdeployAtEnd=true"
      }
      post {
        always {
        junit '**/target/surefire-reports/**/*.xml'
        }
      }
    }
    stage('Integration Tests') {
      when {
        anyOf { branch 'master'; branch 'develop' }
      }
      agent {
        docker {
        image 'maven:3.6.0-jdk-8-alpine'
        args '-v /root/.m2/repository:/root/.m2/repository'
        reuseNode true
        }
      }
      steps {
        sh 'mvn verify -Dsurefire.skip=true'
      }
      post {
        success {
        stash(name: 'artifact', includes: '**/target/*.jar')
        stash(name: 'pom', includes: 'pom.xml')
        // to add artifacts in jenkins pipeline tab (UI)
        archiveArtifacts '**/target/*.jar'
        }
      }
    } 
  }
}