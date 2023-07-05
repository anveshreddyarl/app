import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=68c71e6d-d699-4cb1-ba29-ea17f10c0da2',
        'AZURE_TENANT_ID=c57b7123-8b74-42c6-923a-32c6a13b9725']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'vm2'
      def webAppName = 'myapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'azure', passwordVariable: 'a0a3345b-97f7-4406-9f84-fee1a9c452cc', usernameVariable: 'b6dc1f2b-120e-4d2c-b852-58a9c3f6d2f2')]) {
       sh '''
          az login --service-principal -u b6dc1f2b-120e-4d2c-b852-58a9c3f6d2f2 -p a0a3345b-97f7-4406-9f84-fee1a9c452cc -t c57b7123-8b74-42c6-923a-32c6a13b9725
          az account set -s 68c71e6d-d699-4cb1-ba29-ea17f10c0da2
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package 
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}