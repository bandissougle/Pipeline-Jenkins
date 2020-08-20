pipeline {
  agent any
  environment{
    // This can be nexus3 or nexus2
    NEXUS_VERSION = "nexus3"
    // This can be http or https
    NEXUS_PROTOCOL = "http"
    // Where your Nexus is running. In my case:
    NEXUS_URL = "172.22.100.24:8081"
    // Repository where we will upload the artifact
    NEXUS_REPOSITORY = "maven-snapshots"
    // Jenkins credential id to authenticate to Nexus OSS
    NEXUS_CREDENTIAL_ID = "nexus-credentials"
  }
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
        stash(name: 'pom', includes: '**/pom.xml')
        // to add artifacts in jenkins pipeline tab (UI)
        archiveArtifacts '**/target/*.jar'
        }
      }
    }
    stage('Code Analysis'){
      parallel{
        stage('Next Plugin Analysis'){
          agent {
            docker {
            image 'maven:3.6.0-jdk-8-alpine'
            args '-v /root/.m2/repository:/root/.m2/repository'
            reuseNode true
            }
          }
          steps{
            sh 'mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs'
          }
          post{
            always{
              junit testResults: '**/target/surefire-reports/TEST-*.xml'

              recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
              recordIssues enabledForFailure: true, tool: checkStyle(pattern: '**/target/checkstyle-result.xml')
              recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
              recordIssues enabledForFailure: true, tool: findBugs(pattern: '**/target/findbugsXml.xml')
              recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
            }
          }
        }
        stage('Code Quality Check via SonarQube'){
          steps{
            withSonarQubeEnv("sonarqube-server"){
              sh "/var/jenkins_home/apache-maven-3.6.3/bin/mvn sonar:sonar -Dsonar.host.url=http://172.22.100.22:9000 \
              -Dsonar.login=b43af443b842eda3063651b5c68115cb1a7c6b87"
            }
          }
        }     
      }
    }
    stage('Deploiement des Artefacts sur Nexus'){
      when {
        anyOf { branch 'master' }
      }
      steps{
        script {
          unstash 'pom'
          unstash 'artifact'
          // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
          pom = readMavenPom file: 'pom.xml';
          // Find built artifact under target folder
          filesByGlob = findFiles(glob: '**/target/*.jar');
          // Print some info from the artifact found
          echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
          echo "${filesByGlob[1].name} ${filesByGlob[1].path} ${filesByGlob[1].directory} ${filesByGlob[1].length} ${filesByGlob[1].lastModified}"
          echo "${filesByGlob[2].name} ${filesByGlob[2].path} ${filesByGlob[2].directory} ${filesByGlob[2].length} ${filesByGlob[2].lastModified}"
          echo "${filesByGlob[3].name} ${filesByGlob[3].path} ${filesByGlob[3].directory} ${filesByGlob[3].length} ${filesByGlob[3].lastModified}"
          echo "${filesByGlob[4].name} ${filesByGlob[4].path} ${filesByGlob[4].directory} ${filesByGlob[4].length} ${filesByGlob[4].lastModified}"
          echo "${filesByGlob[5].name} ${filesByGlob[5].path} ${filesByGlob[5].directory} ${filesByGlob[5].length} ${filesByGlob[5].lastModified}"
          echo "${filesByGlob[6].name} ${filesByGlob[6].path} ${filesByGlob[6].directory} ${filesByGlob[6].length} ${filesByGlob[6].lastModified}"

          // Extract the path from the File found
          artifactPath = filesByGlob[0].path;
          // Assign to a boolean response verifying If the artifact name exists
          artifactExists = fileExists artifactPath;
          if (artifactExists) {
            nexusArtifactUploader(
            nexusVersion: NEXUS_VERSION,
            protocol: NEXUS_PROTOCOL,
            nexusUrl: NEXUS_URL,
            groupId: pom.groupId,
            version: pom.version,
            repository: NEXUS_REPOSITORY,
            credentialsId: NEXUS_CREDENTIAL_ID,
            artifacts: [
              // Artifact generated such as .jar, .ear and .war files.
              [artifactId: pom.artifactId,
              classifier: '',
              file: artifactPath,
              type: "jar"
              ]
            ]
            )
          } else {
            error "*** File: ${artifactPath}, could not be found";
          }
        }
      }
    }
  }
}