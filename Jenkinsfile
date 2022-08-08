pipeline {
  agent any  
  
  environment {                
         SONAR_HOME =  tool name: 'sonar-scanner'
    }

  stages {
         
    stage('Scan SAST') {
      steps {
         withSonarQubeEnv('sonarqube-server') { 
             

            sh 'echo export SONAR_HOME= "{env.$SONAR_HOME}"'   
            sh '''
        
                    export PATH=$PATH:$SONAR_HOME/bin

                      sonar-scanner \
                      -Dsonar.projectKey=jskey \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://192.168.100.115:9000 \
                      -Dsonar.login=b30981b3ff3768305a8f05253b95886320794b0f
              '''
        
        }
      }
    }
   stage('Dependency Check') {
    steps {
        dependencyCheck additionalArguments: ''' 
            -o "./" 
            -s "./"
            -f "ALL" 
            --prettyPrint''', odcInstallation: 'Dependency-Check'
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
          }    
    }
    
    stage('Wait Quality Gate'){
            steps {
                script {
                    withSonarQubeEnv("sonarqube-server"){
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            timeout(time: 1, unit: 'HOURS') {
                                waitForQualityGate abortPipeline: false
                            }
                        }
                    }
                }
            }
        }
    
    stage('Send report to DefecDojo') {
        steps {
           
            sh 'cp ../upload-files.py .'
            sh 'chmod +x upload-files.py'
            sh "python3 upload-files.py --result_file /var/lib/jenkins/workspace/Pipeline-Sec/dependency-check-report.xml  --scanner 'Dependency Check Scan' --host 127.0.0.1:8080 --api_key d64dca4d31e577b7924ac6e0b8cc59f4b1526430  --name Namerepository"
            sh "sonar-report --sonarurl='http://192.168.100.115:9000'  --sonarcomponent='jskey' --project='jskey Project'  -sonartoken='02a034674a8cc59121f69d82c5f5b6d2ece15da7'  --sinceleakperiod='false' --fixMissingRule='true'  --noSecurityHotspot='true'  --allbugs='true' > sonar.html"         
            sh "python3 upload-files.py --result_file /var/lib/jenkins/workspace/Pipeline-Sec/sonar.html  --scanner 'SonarQube Scan' --host 127.0.0.1:8080 --api_key d64dca4d31e577b7924ac6e0b8cc59f4b1526430  --name Namerepository"
        } 
            
    }
    //   stage('Subindo o container novo') {
    //             steps {
    //                 script {
    //                     try {
    //                         sh 'docker run -d -p 8989:8989 node-goof:latest'
    //                     } catch (Exception e) {
    //                         sh "echo $e"
    //                         currentBuild.result = 'ABORTED'
    //                         error('Erro')
    //                 }
    //             }
    //         }
    //     }
  }

}
