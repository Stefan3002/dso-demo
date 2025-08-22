pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Static Analysis') {
      parallel {
        stage('SCA') {
    steps {
        container('maven') {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }
    }
    post {
        always {
            archiveArtifacts(
                allowEmptyArchive: true,
                artifacts: 'target/dependency-check-report.html',
                fingerprint: true,
                onlyIfSuccessful: true
            )
            // Optional: publish the Dependency-Check report inside Jenkins UI
            // dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
    }
}

        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
   stage('Package') {
    parallel {
        stage('CreateJarfile') {
            steps {
                container('maven') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        stage('OCIImageBnP') {
            steps {
                container('kaniko') {
                    sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/stefan913/jenkins-test'
                }
            }
        }
    }
}


    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
