pipeline{
    agent any
    triggers {
      pollSCM '* * * * *'
    }
    stages{
        stage("Maven & Sonar Parallel"){
            parallel
            {

             stage("Maven Build"){
     when {
         expression{
         env.BRANCH_NAME.equals("develop") 
             //|| env.BRANCH_NAME.startsWith("feature")
         }
     }
     steps{
        sh "mvn package"
     }
 }
 
  stage("SonarQube Analysis"){
      when {
          branch "develop1"
      }
      steps{
          script{
         withSonarQubeEnv('sonar7') {
             sh "/opt/sonar-scanner/bin/sonar-scanner"
              //sh "mvn sonar:sonar"
             
         }
          }
      }
  }

          
        }
    }
       
      
          stage("SonarQube Status"){
            when {
                branch "develop1"
            }
            steps{
              
               timeout(time: 1, unit: 'HOURS') {
                     script{
                //    For this to work, we should add webhook in sonar
                //    http://172.31.3.50:8080/sonarqube-webhook/
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
                }
            }
        }

        
        
        stage("Nexus"){
            when {
                branch "develop"
            }
            steps{
               echo "uploading artifacts to nexus...."
                 nexusArtifactUploader artifacts: [[artifactId: 'myweb', classifier: '', file: 'target/myweb-0.0.6.war', type: 'war']], credentialsId: 'nexus3', groupId: 'in.javahome', nexusUrl: '172.31.2.74:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myapp-release', version: '0.0.6'
            }
        }
        stage("Deploy To QA"){
            when {
                branch "qa"
            }
            steps{
               echo "deploying to qa server...."
            }
        }
        stage("Deploy To production"){
            when {
                branch "master"
            }
            steps{
               echo "deploying to master server...."
            }
        }
    }
}
