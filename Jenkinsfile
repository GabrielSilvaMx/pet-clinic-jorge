node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    def mvn = tool 'maven-3.8.6';
    withSonarQubeEnv() {
      sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=PetClinic"
    }
  }
}

pipeline {

    agent any
    
    stages {
        
        stage('Build') {
            steps {
                echo 'Build'
                sh "mvn --batch-mode package" 
            }
        }

        stage('Archive Unit Tests Results') {
            steps {
                echo 'Archive Unit Test Results'
               step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/TEST-*.xml'])
            }
        }
        
        stage('Publish Unit Test results report') {
            steps {
                echo 'Report'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: false, reportDir: 'target/sonar/', reportFiles: 'index.html', reportName: 'jacoco report', reportTitles: ''])

             }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploy'
                sh '''
                    for runName in `docker ps | grep "alpine-petclinic" | awk '{print $1}'`
                    do
                        if [ "$runName" != "" ]
                        then
                            docker stop $runName
                        fi
                    done
                    docker build -t alpine-petclinic -f Dockerfile.deploy .
                    docker run --name alpine-petclinic --rm -d -p 9966:8080 alpine-petclinic 
                '''
            }
        }
    }
}
