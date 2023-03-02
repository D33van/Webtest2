pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('JAVA') {
    steps{
    sh 'java -version' 
    }
  }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar-8.9.9') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war justdial@172.29.87.55:/opt/tomcat/webapps/webapp.war'
              }      
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no justdial@172.29.87.55 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://172.29.87.55:8888/webapp/" || true'
        }
      }
    }
    
  }
}
