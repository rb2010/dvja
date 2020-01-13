pipeline {
  agent any

  tools {
    maven "apache-maven-3.6.3"
  }

  stages {
    stage('Build') {
      steps {
        git 'https://github.com/ajlanghorn/dvja.git'
        sh "mvn clean package"
      }
    }
    stage('Check dependencies') {
      steps {
        dependencyCheck additionalArguments: '', odcInstallation: 'Dependency-Check'
        dependencyCheckPublisher pattern: ''
      }
    }
    stage('Scan for vulnerabilities') {
    steps {
        sh 'java -jar dvja-*.war && zap-cli quick-scan --self-contained --spider -r http://127.0.0.1 && zap-cli report -o zap-report.html -f html'
     }
    }
    stage('Analysis') {
      steps {
        sh "mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs"
      }
    }
    stage('Publish to S3') {
      steps {
        sh "aws s3 cp target/dvja-1.0-SNAPSHOT.war s3://cicdsecops-buildartifacts-j03rfel4p8ns/dvja-1.0-SNAPSHOT.war"
      }
    }
    stage('Tidy up') {
      steps {
        cleanWs()
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'zap-report.html', fingerprint: true
      recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
      recordIssues enabledForFailure: true, tool: checkStyle()
      recordIssues enabledForFailure: true, tool: spotBugs()
      recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
      recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
    }
    }
}
}
