pipeline {
    agent any
    stages {

      stage('clean and build')
            {
                steps
                 { 
                    sh 'mvn clean package'
                 }
            }
           stage("SonarQube analysis") {
            steps {
              withSonarQubeEnv('sonarqube') {
                sh 'mvn sonar:sonar -Pprofile1'
                  input('Do you want to proceed?')
              }
            } 
            } 
        
        stage("Quality Gate") {
            steps {
              timeout(time: 5, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
                waitForQualityGate abortPipeline: true //waiting for a task to be completed
              }
            }
       }
  stage("nexus") {
            steps {
          withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'password', usernameVariable:'username')]) {
              sh 'curl -u ${username}:${password} --upload-file target/WebApplication-1.war http://ec2-18-224-155-110.us-east-2.compute.amazonaws.com:8081/nexus/content/repositories/devopstraining/phoenixTeam/WebApplication-${BUILD_NUMBER}.war'
              sh 'curl -u ${username}:${password} --upload-file target/WebApplication-1.war http://ec2-18-224-155-110.us-east-2.compute.amazonaws.com:8081/nexus/content/repositories/devopstraining/phoenixTeam/WebApplication-1.war'
        //sh 'curl -v -F r=devopstraining -F hasPom=true -F e=war -F file=@pom.xml -F file=@target/WebApplication-0.0.1-SNAPSHOT.war -u ${username}:${password} http://ec2-18-224-155-110.us-east-2.compute.amazonaws.com:8081/nexus/content/repositories/devopstraining/phoenixTeam/WebApplication-0.0.1-SNAPSHOT.war'
         }
        }
        }
        
        //stage(" copying war to ansible"){
            //steps{
               // withCredentials([string(credentialsId: 'ansible_credentials', variable: 'password')]){
                   // sh 'sshpass -p ${password} scp -v -o StrictHostKeyChecking=no target/WebApplication-1.war ansadmin@172.31.10.200:/opt/playbooks/target/'
                //}
               //sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible_server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//playbooks', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
        //}
   // }
        stage("Deploy to Tomcat"){
            steps{
                withCredentials([string(credentialsId: 'ansible_credentials', variable: 'password')]){
                    sh 'sshpass -p ${password} ssh -o StrictHostKeyChecking=no ansadmin@172.31.10.200 -C \"ansible-playbook /opt/playbooks/download.yml\"'
                }
            //sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible_server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook /opt/playbooks/copyfile.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
            
            }
            
post {
    always {
           jiraSendDeploymentInfo environmentId: 'jiradev', environmentName: 'jiraenvdev', environmentType: 'development', site: 'varshi26.atlassian.net'
       }
   }
        }
    }
        post{
                success{
                    //slackSend baseUrl: 'https://hooks.slack.com/services/', botUser: true, channel: 'team_phoenix', color: 'good', message: "build number is '[${BUILD_NUMBER}]'", notifyCommitters: true, username: 'phoenix', tokenCredentialId: 'slack-cred'
                    slackSend baseUrl: 'https://hooks.slack.com/services/', channel: 'team_phoenix', color: 'good', message: "build with number '[${BUILD_NUMBER}]' IS SUCCESSFUL", tokenCredentialId: 'slack-cred', username: 'phoenix'
                        }
                unsuccessful{
                    slackSend baseUrl: 'https://hooks.slack.com/services/', channel: 'team_phoenix', color: 'danger', message: "build with number '[${BUILD_NUMBER}]' IS A FAILURE", tokenCredentialId: 'slack-cred', username: 'phoenix'
                            }
           /* withCredentials([string(credentialsId: 'slack-cred', variable: 'slack_credentials')])
            {
            curl -H "Content-type: application/json" -X POST --data-urlencode -d 
"payload='{
"username": "phoenix",
"attachments": [
    {
        "color": "good",
        "fields": [
            {
                "title": "success",
                "value": "$BUILD_NUMBER",
                "short": false
            }
        ]
    }
    ]
            }'" https://hooks.slack.com/services/${slack_credentials}
        }*/
}
}
