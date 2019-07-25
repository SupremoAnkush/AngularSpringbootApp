pipeline 
{
    agent any
    
         /*triggers{
            PollSCM('* * * * *')
         }*/
    
    
    tools {nodejs "nodejs"}
    
    
    
    stages {
        stage('git cloning') {
            steps {
               sh 'rm -rf dist ankushApp.zip'
               git credentialsId: 'AnkushGithub', url: 'https://github.com/SupremoAnkush/AngularSpringbootApp.git'
                
            }
        }
        stage('installing dependencies') {
            steps {
               sh 'npm install'
            }
        }
        //  stage('test') {
        //      steps {
        //          sh 'npm run test'
        //      }
        // }
        stage('SonarQube') 
       {
           
            environment {
                scannerHome=tool 'sonar scanner'
            }
             //tools {scannerHome "SonarScanner"}
        steps{
             withSonarQubeEnv(credentialsId: 'sonar_token_ankush', installationName: 'sonar_server') {
                  sh '${scannerHome}/bin/sonar-scanner -Dproject.settings=./sonar-project.properties'
              }
              //sh 'npm run sonar'
           
            //timeout(time: 1, unit: 'HOURS') {
                //waitForQualityGate abortPipeline: true
            //}
           }
            
        }          
      
       
        stage('build') {
            steps {
                sh 'npm run build'
                
            }
        }
        stage ('zipping'){
            steps {
                 
                sh 'cd dist/AngularSpringbootApp;ls;zip -r ../../ankushApp.zip . ; ls;'
            }
        }
        stage ('Uploading artifact to nexus'){
            steps{
                sh 'ls'
                withCredentials([usernamePassword(credentialsId: 'ankush_nexus_key', passwordVariable: 'ankush_nexus_password', usernameVariable: 'ankush_nexus')]) {
                 sh label: '', script: 'curl -v -u ${ankush_nexus}:${ankush_nexus_password} --upload-file ankushApp.zip http://3.14.251.87:8081/nexus/content/repositories/devopstraining/ankushApp/ankush-${BUILD_NUMBER}.zip'
                 
                }
               
            }
        }
      
        stage ('Deploy') {
            steps {
              withCredentials([file(credentialsId: 'angular-react-deployment-server', variable: 'deployment_server')]) {
                   sh 'scp -v -i ${deployment_server} ankushApp.zip ubuntu@18.224.182.74:/home/ubuntu'
                   sh 'ssh -v -i ${deployment_server} ubuntu@18.224.182.74 "cd /home/ubuntu;unzip -o ankushApp.zip . ;ls;"'
               }
            }
        }
        
    }
    post { 
         success { 
            echo 'notified to slack'
            slackSend (color: '#00FF00', message: " JOB SUCCESSFUL: Job '${JOB_NAME} [${BUILD_NUMBER}]' (${BUILD_URL})")
         }
         failure {
            echo 'notified to slack'
            slackSend (color: '#FF0000', message: " JOB FAILED: Job '${JOB_NAME} [${BUILD_NUMBER}]' (${BUILD_URL})")
         }
    }

}

