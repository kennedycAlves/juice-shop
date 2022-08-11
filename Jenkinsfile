pipeline {
  agent any  
  
  environment {                
         SONAR_HOME =  tool name: 'sonar-scanner'
         IDCONTAINER = ''
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
    
    stage('Down old containers') {
        steps {
            script {
                try {
                    sh """
                        docker rm -f owasp-scan
                        docker rm -f sec-scan
                    """
                } catch (Exception e) {
                    sh "echo $e"
                }
            }
        }
    }   
        
    stage('Docker Build') {
      agent any
        steps {
            sh 'sudo docker build -t sec/test-dinamic:latest .'
        }
    }
    stage('Up Docker Image') {
      agent any
        steps {
            
            sh 'sudo docker run -d --name sec-scan -p 3000:3000 sec/test-dinamic'
        }
    }
    
        
        stage('Scan OWASP ZAP App container') {
             steps {
                 script {
                         echo "Pulling up last OWASP ZAP container --> Start"
                         sh 'sudo docker pull owasp/zap2docker-stable'
                         echo "Pulling up last VMS container --> End"
                         echo "Starting container --> Start"
                         
                         
                         sh """
                                sudo docker run -dt --name owasp-scan \
                                owasp/zap2docker-stable  /bin/bash 
                              
                            """
                       
                            
                         sh 'sudo docker exec owasp-scan mkdir /zap/wrk'
                         
						 sh """
						    sleep 1m
                            sudo docker exec owasp-scan zap-baseline.py \
                            -t http://192.168.100.115:3000/ \
                            -x report-zap.xml -I &&
                            myvar=\$(sudo docker ps -aqf 'name=owasp-scan') 
                            echo export myvar 
                            sudo docker cp \$myvar:/zap/wrk/report-zap.xml \${WORKSPACE}/report-zap.xml
    						 
                         """
                    
                 }
             }
          }  
             
    
    stage('Send report to DefecDojo') {
        steps {
             script {
                sh 'cp ../upload-files.py .'
                sh 'chmod +x upload-files.py'
                sh "python3 upload-files.py --result_file /var/lib/jenkins/workspace/Pipeline-Sec/dependency-check-report.xml  --scanner 'Dependency Check Scan' --host 127.0.0.1:8080 --api_key d64dca4d31e577b7924ac6e0b8cc59f4b1526430  --name Namerepository"
                sh "sonar-report --sonarurl='http://192.168.100.115:9000'  --sonarcomponent='jskey' --project='jskey Project'  -sonartoken='02a034674a8cc59121f69d82c5f5b6d2ece15da7'  --sinceleakperiod='false' --fixMissingRule='true'  --noSecurityHotspot='true'  --allbugs='true' > sonar.html"         
                sh "python3 upload-files.py --result_file /var/lib/jenkins/workspace/Pipeline-Sec/sonar.html  --scanner 'SonarQube Scan' --host 127.0.0.1:8080 --api_key d64dca4d31e577b7924ac6e0b8cc59f4b1526430  --name Namerepository"
                sh "python3 upload-files.py --result_file /var/lib/jenkins/workspace/Juice-Shop/report-zap.xml  --scanner 'ZAP Scan' --host 127.0.0.1:8080 --api_key d64dca4d31e577b7924ac6e0b8cc59f4b1526430  --name Namerepository"
 
            
           
                try {
                    sh """
                        docker rm -f owasp-scan
                        docker rm -f sec-scan
                    """
                } catch (Exception e) {
                    sh "echo $e"
                }
            }
            
        } 
            
    }
    
  }

}
