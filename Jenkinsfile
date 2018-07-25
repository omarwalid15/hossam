import groovy.json.JsonSlurper
import java.text.SimpleDateFormat;
import java.io.File
import java.util.Date
import java.util.Calendar
def ReqID = ''
String Create = '''{
     "root":{
        "CreateCR":{
        "xmlns":"http://xmlns.vfe.com.eg/DevOps/celfocus/Client/CreateCR/CreateCR/request",
        "Template_Name__c":"Tibco Commercial Release_DevOps",
        "Asset_ID__c":"%s",
        "Short_Description":"%s",
        "User_Name__c":"%s",
        "PipelineName":"%s", 
        "PipelineBuildID":"%s",
        "Webhook":"%s",
        "Schduled_Start_Time":"%s",
        "Schduled_End_Time":"%s",
        "Requester_Manager":"%s",
        "Task_Implementation":"%s"
        }
  }
}
'''
String email = '''CRQ has been created. Use the following parameters for deployment: 
ReqID= %s
Application = %s 
Developer = %s
Description = %s'''


pipeline {
    agent {
    node {
        label 'master'
    }
} 

    stages {
        stage('Build') { 
            steps { 
            build job: 'PRD_Auto_Build', parameters: [string(name: 'Applications', value: "${Application}")]
            echo "Building..."
            }
        }
        
        stage ('Create CR') {
            steps {
                script{
                    String Directory = "${Application}".substring( 0, "${Application}".indexOf("/"))
                    echo Directory
                //    String Requester =  sh script: ''' cd /devauto/ci/jenkins/jobs/PRD_Auto_Build/workspace/CRM
                  //      git log --no-merges --pretty=format:"%ce" -1 ''' + Directory, returnStdout: true
                //    String Approver = sh script: ''' cd /devauto/ci/jenkins/jobs/PRD_Auto_Build/workspace/CRM
                  //       git log --merges --pretty=format:"%ce" -1 ''' + Directory, returnStdout: true
                   String Requester = "Mohamed.Bastawisy@vodafone.com"
                    String Approver = "Mohamed.SeragElDin-Shaker@vodafone.com"
                    echo Requester
                    echo Approver
                    Date date = new Date() + 1
                    String datePart = date.format("yyyy-MM-dd")
                    String Start_Time = datePart + "T" + "07:30:00" +"+02:00"
                    String End_Time = datePart + "T" + "07:45:00" +"+02:00"
                    echo Start_Time
                    echo End_Time
                    hook = registerWebhook()
                    String output = String.format(Create,"${Application}", "${Description}",Requester,"${JOB_NAME}","600","${hook.getURL()}",Start_Time,End_Time,Approver,"${Implementation}")
                    echo output
                    httpRequest acceptType: 'APPLICATION_JSON_UTF8', consoleLogResponseBody: true, contentType: 'APPLICATION_JSON', httpMode: 'POST', outputFile: 'CRQ Response', requestBody: output, responseHandle: 'NONE', url: 'http://10.230.92.218:19001/CRQ/new', validResponseCodes: '100:500'
                    fileExists 'CRQ Response'
                    filename = readFile 'CRQ Response'
                    def parser = new JsonSlurper()
                    def jsonResp = parser.parseText(filename)
                    ReqID = jsonResp.root.RequestID   
                    def ecode = jsonResp.root.eCode
                    if (ecode != '0') { error '"CRQ Create Failed"' }
                    echo (ReqID)
                }
            }
        }
    
        stage ('Wait for Approval') 
        {
            steps {
                script {
				wrap([$class: 'BuildUser']) 
				{
                    data = waitForWebhook hook
                    echo data
                    def parser = new JsonSlurper()
                    def jsonResp = parser.parseText(data)
                    def CRQStatus = jsonResp.root.CRQ_Status
                    if (CRQStatus != 'Scheduled') { error '"CRQ Rejected"' }
                    CRQ = jsonResp.root.CRQ_ID

					email = String.format(email,ReqID,"${Application}",env.BUILD_USER,"${Description}")
				}
                
				}
                   mail body: email, cc: 'TechIT-CRM-Integration@voda.com', subject: 'CRQ Created - Application Scheduled for Deployment ', to: 'TechIT-CRM-Integration-ProvisioningSupport@voda.com'
                  // mail body: email, subject: 'CRQ Created - Application Scheduled for Deployment ', to: 'hazem.abdo@vodafone.com'
                  slackSend channel: '#deployments', color: 'good', message: email, teamDomain: 'tibco-operations', token: 'MqqVhEIXmoFO55kLWfJjb9mf'
               
            }
        }
    }  
}