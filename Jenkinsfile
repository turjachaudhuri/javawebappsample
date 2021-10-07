import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node('slave') {
  withEnv(['AZURE_SUBSCRIPTION_ID=d0aae499-f910-4d0c-a3e7-2986a6f02c82',
        'AZURE_TENANT_ID=e00783c2-65c4-4231-8e2b-6aaf165c4de8']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
   
   stage('test') {
      // run your test / unit tests here
    }
	
    stage('deploy') {
      def resourceGroup = 'jenkins-demo-mtech'
      def webAppName = 'jenkinsdevopsassignment'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'jenkinsServicePrincipal', passwordVariable: 'ZEGm3i9afe6BMfA-g-awrL6cne79jrfl8F', usernameVariable: '80f095a5-5ea1-47aa-8984-d7237efa471d')]) {
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
