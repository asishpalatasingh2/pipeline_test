pipeline {
    agent any
    
    environment {
        GIT_REPO = 'https://github.com/asishpalatasingh2/pipeline_test.git'
	ANYPOINT_CREDENTIALS = credentials('MuleCredential') 
        APPROVAL_EMAIL = 'asish.tgh@gmail.com'
        JENKINS_URL = 'http://localhost:8083/'
	} 
    
    stages {
		stage('Build Application') { 
            steps { 
                bat 'mvn clean install -DskipTests'
            } 
        }
		stage('Test') { 
            steps { 
                bat 'mvn test'
            } 
        }
        stage('Approval') {
            steps {
                script {
                	if(env.GIT_BRANCH.toLowerCase() == 'origin/main') {
                    	sendApprovalEmail()
                    	def approvalResponse = input(
                        	id: 'userInput',
                        	message: 'Please approve the deployment to dev environment',
                        	parameters: [
                            	[$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Approve Deployment?', name: 'Approve']
                        	],
                        	timeout: 5 * 60, // 5 minutes timeout
                        	timeoutMessage: 'Approval timed out. Deployment not approved.'
                    	)
                    	if (!approvalResponse) {
                        	error 'Deployment not approved. Aborting.'
                    	}
                    } else{
                    echo "Approval Not needed"
                    }
                }
            }
        }
        stage('Build and Deploy') {
            steps {
                script {
					if(env.GIT_BRANCH.toLowerCase() == 'origin/develop') {
                    deployToAnypoint('Dev','b-pipeline-dev')
                    }
                    else if(env.GIT_BRANCH.toLowerCase() == 'origin/qa') {
                    deployToAnypoint('QA','b-pipeline-qa')
                    }
                    else if(env.GIT_BRANCH.toLowerCase() == 'origin/main') {
                    deployToAnypoint('Sandbox','b-pipeline-prod')
                    }
                    else {
                    echo "Branch not configured"
                    echo env.GIT_BRANCH
                    
                    }
                }
            }
        }
    }
post {
    	success {
            mail to: 'asish.tgh@gmail.com',
            subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Good news! The Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} was successful.\n\nCheck it out here: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'asish.tgh@gmail.com',
            subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Unfortunately, the Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.\n\nCheck it out here: ${env.BUILD_URL}"
        }
    }
}

def deployToAnypoint(environmentName,appName) {
	echo environmentName
    echo 'Deploying mule project due to the latest code commit…' 
	echo 'Deploying to the configured environment….' 
	bat """
        mvn clean deploy -DmuleDeploy \
            -DPLATFORM_USERNAME=${env.ANYPOINT_CREDENTIALS_USR} \
            -DPLATFORM_PASSWORD=${env.ANYPOINT_CREDENTIALS_PSW} \
            -Dmule_env=${environmentName} \
            -Dmule_appName=${appName}
    """
    }
def sendApprovalEmail() {
    def subject = "Approval Required: Jenkins Pipeline Deployment to Dev Environment"
    def body = """
        <html>
        <body>
        <p>Dear Approver,</p>
        <p>A deployment to the dev environment is pending your approval.</p>
        <p>Please approve or deny using the Jenkins input step in the build page.</p>
        <p>Branch: ${env.BRANCH_NAME}</p>
        <p>View the build at: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
        <p>Thank you.</p>
        </body>
        </html>
    """
    mail to: "${APPROVAL_EMAIL}", subject: subject, body: body, mimeType: 'text/html'
}
