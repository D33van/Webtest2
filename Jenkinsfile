pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        bat '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
   stage ('Check-Git-Secrets') {
     steps {
      bat 'rm trufflehog || true'
       bat 'docker run gesellix/trufflehog --json https://github.com/D33van/Webtest2.git > trufflehog'
        bat 'cat trufflehog'
     }
    }
  
    stage ('Source Composition Analysis') {
      steps {
         bat 'rm owasp* || true'
         bat 'wget "https://raw.githubusercontent.com/cehkunal/webapp/master/owasp-dependency-check.sh" '
         bat 'chmod +x owasp-dependency-check.sh'
         bat 'bash owasp-dependency-check.sh'
         bat 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          bat 'mvn sonar:sonar'
          bat 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      bat 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                bat 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@13.232.202.25:/prod/apache-tomcat-8.5.39/webapps/webapp.war'
              }      
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         bat 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.158.44 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://domain:8080/webapp/" || true'
        }
      }
    }
    
  }
}
