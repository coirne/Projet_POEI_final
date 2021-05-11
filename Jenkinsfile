pipeline {
    agent any
    stages {
        stage('Clone Git repo') {
        steps {
            git branch: 'main', url: 'https://github.com/coirne/Projet_POEI_final.git'
              }
        }
        
        stage('Test : pytest'){        
        steps {
                sh '''
                    pip3 install -r requirements.txt
                    cd tests
                    pytest
                '''
          }
        }
        
        stage('Code Review : Sonarqube') {
        steps {
        script {
           def scannerHome = tool 'SonarQube Scanner';
               withSonarQubeEnv('SonarQube'){
               sh " ${tool("SonarQube Scanner")}/bin/sonar-scanner \
               -Dsonar.projectKey=Projet_CI \
               -Dsonar.sources=. \
               -Dsonar.css.node=. \
               -Dsonar.host.url=http://172.27.208.1:9000 \
               -Dsonar.login=754fee56c1f8a22d2e88cbadf675a212d7fa68fb"
                   }
                }
            }
        }
    
        stage('Store Artefact : nexus'){        
        steps {
                sh '''
                    rm -f /var/jenkins_home/workspace/*.tar.gz
                    tar -czf archive_version$BUILD_NUMBER.tar.gz /var/jenkins_home/workspace/Projet_POEI_final/
                    curl -v -u admin:corine --upload-file /var/jenkins_home/workspace/archive_version$BUILD_NUMBER.tar.gz  http://172.27.208.1:8081/repository/Projet_POEI_final/
                '''
          }

    }
}  
}
