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
                "target": "libs-snapshot-local/war/"
               }
           ]
        }"""
        tag = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED,"yyyyMMdd"}-${BRANCH_NAME}-${BUILDS_TODAY}-${BUILD_NUMBER}');
    }
    
    stages {
        try{
            stage ('Initialize') {
                steps {
                        sh '''
                            echo "PATH = ${PATH}"
                            echo "M2_HOME = ${M2_HOME}"
                        '''	
                }
            } 
            stage ('Build Stage') {
                steps {
                    sh 'mvn clean install -DskipTests'
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
                }
            }
        }
    
    catch (error) {
       stage ("Cleanup after fail")
       emailext attachLog: true, body: "Build failed (see ${env.BUILD_URL}): ${error}", subject: "[JENKINS] ${env.JOB_NAME} failed", to: 'someone@example.com'
       throw error
   } finally { 
        always { 
            cleanWs()
    
        }
   }
    }
}
