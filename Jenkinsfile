import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=813c9613-a0ca-4f6a-9900-8857b8134e62',
        'AZURE_TENANT_ID=b02cc036-affc-49c8-9e5f-aea56cd0ef16']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'SjhaQuickstartJenkins-rg'
      def webAppName = 'sjhaJavaApp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '738b47c3-1de6-46cd-b72e-321f393a3056', passwordVariable: 'RHXdsRY6b14StYWXcJ.5bR4AdELfnEo~3~', usernameVariable: '738b47c3-1de6-46cd-b72e-321f393a3056')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
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
