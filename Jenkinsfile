node
{
    echo "git branch name: ${env.JOB_NAME}"
   echo "build number is: ${env.BUILD_NUMBER}"
   echo "node name is: ${env.NODE_NAME}"
try{
    //stage-1 git checkout
    stage ('git checkout')
    {
        git branch: 'development', url: 'https://github.com/vaasvik/Web-App-project.git'
    }
    //stage-2 build the project & createpackage
    def mavenHome=tool name: 'Maven 3.9.9'
    stage ('build')
    {
        sh "${mavenHome}/bin/mvn clean package"
    }
    //stage-3 generating sonar report
    stage('code quailty report')
    {
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    //stage-4 upload to nexus
    stage('upload to nexus')
    {
        sh "${mavenHome}/bin/mvn clean deploy"
    }
    //stage-5 deploy war to tomcat
    stage ('deploying war file in tomcat')
    {
        echo "deploying war file using curl"
        sh """
        curl -u admin:sai@123 \
        --upload-file /var/lib/jenkins/workspace/script-pipeline/target/maven-web-application.war \
        "http://3.230.200.8:9090/manager/text/deploy?path=/maven-web-application&update=true"
        """
    }
}
catch (e) 
  {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } 
  finally
  {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}// node end
def notifyBuild(String buildStatus = 'STARTED') 
{
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') 
  {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } 
  else if (buildStatus == 'SUCCESS') 
  {
    color = 'GREEN'
    colorCode = '#00FF00'
  } 
  else 
  {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#jio-dev')
  slcakSend (color: colorCode, message: summary, channel: '#jio-project')
}
