def getReportZipFile() {
    return  VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED,"yyyyMMdd"}-${BRANCH_NAME}-${BUILDS_TODAY}-${BUILD_NUMBER}.zip')
}

def getEmailRecipients() {
    return ''
}

pipeline {
   agent {
    	node {
        	label 'Java'
        	customWorkspace '/home/mchathur/Jenkins/'
   	    }
    } 

    environment {
        registry = "sahan89/hello-world-app"
        registryCredential = 'dockerhub'
        dockerImage = ''
        def uploadSpec = """{
           "files": [
               {
               "pattern": "target/*.war",
                "target": "generic-local/sahan/DevOpsHelloWorldApp/war/"
               }
           ]
        }"""
        tag = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED,"yyyyMMdd"}-${BRANCH_NAME}-${BUILDS_TODAY}-${BUILD_NUMBER}');
    }

    parameters {
        string(defaultValue: "$emailRecipients",
                description: 'List of email recipients',
                name: 'EMAIL_RECIPIENTS')
    }
    stages {
            stage ('Initialize') {
                steps {
                    timestamps {
                      logstash{
                        sh '''
                            echo "PATH = ${PATH}"
                            echo "M2_HOME = ${M2_HOME}"
                        '''	
                      }
                    }
                }
            } 
            stage ('Build Stage') {
                steps {
                    script {

                    try{
                        sh 'mvn clean install -DskipTests'
                    }
                    catch (error) {
                    stage ("Cleanup after fail")
                    emailext attachLog: true, body: "Build failed (see ${env.BUILD_URL}): ${error}", subject: "[JENKINS] ${env.JOB_NAME} failed", to: 'someone@example.com'
                    throw error
                    } 
                } 
                }             
            }

            stage ('Deploy Stage') {
                    steps {
                            sh '''
                            mv target/DevOpsHelloWorldApp.war target/hello-world-app.${tag}.war
                            set BUILD_NUMBER=${tag}
                            '''
                    }
            }
            stage('Upload Artifact') {
                    steps {
                        script {
                                def server = Artifactory.server 'Artifactory-1'
                                def buildInfo = server.upload spec: uploadSpec
                        }     
                    }
            }
            stage ('Build Docker Image Stage') {
                    steps {
                        script {
                            dockerImage = docker.build registry + ":$tag"
                            }
                }
            }
            stage('Deploy Docker Image Stage') {
                    steps{
                        script {
                            docker.withRegistry( '', registryCredential ) {
                                dockerImage.push()
                            }
                        }
                    }
            }
            stage('Archive Artifacts'){
                steps{
                    archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
                    zip dir: "${workspace}/target/maven-archiver/", zipFile: "$reportZipFile" // Create a zip file of content in the workspace
                }
            }
        }
    
    post {
        // Run regardless of the completion status of the Pipeline run
        always {
            // send email
            // email template to be loaded from managed files
            emailext body: '${SCRIPT,template="managed:EmailTemplate"}',
                    attachLog: true,
                    compressLog: true,
                    attachmentsPattern: "$reportZipFile",
                    mimeType: 'text/html',
                    subject: "Pipeline Build ${BUILD_NUMBER}",
                    to: "${params.EMAIL_RECIPIENTS}"

            // clean up workspace
            deleteDir()
        }
    }
}
