import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=f5960a43-d4d0-4bd1-9bd1-5c71e4a8f2f4',
           'AZURE_TENANT_ID=34457269-fdf3-4c18-8d2d-7c9354d2c2b9']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'ZTYikesapp'
      
      // Azure 登录
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      
      // 获取发布设置
      def pubProfilesJson = sh(script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true).trim()
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      
      if (ftpProfile) {
        // 上传应用包
        sh "az webapp deploy --resource-group $resourceGroup --name $webAppName --src-path target/calculator-1.0.war --type war"
      } else {
        error "FTP publish profile not found."
      }
      
      // Azure 登出
      sh 'az logout'
    }
  }
}
