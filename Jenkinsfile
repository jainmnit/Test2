def artifactVer = ""
def pomArtifactId = ""
pipeline {
	agent any
		tools{
			maven 'my-maven'
	    }
		environment
		{ 
          _JAVA_OPTIONS="-Xmx2048M -XX:MaxPermSize=1024m"
          SONAR_RUNNER_OPTS="-Xms2048m -Xmx2048m -XX:MaxPermSize=2048m"
          NEXUS_VERSION = "nexus3"
          NEXUS_PROTOCOL = "http"      
        }		
	    stages {
		    stage("Initialization") {
                             steps {
				   script{
			         pom = readMavenPom(file: 'pom.xml')
				 pomArtifactId = pom.getArtifactId().toString()
			         def tmpversion = "${pom.version}"
			         tmpversion = tmpversion.replaceAll('-\\$\\{buildNumber\\}','')
				 artifactVer = tmpversion + "-" + currentBuild.number  
				 print("tmpversion"+tmpversion)
				  
				 currentBuild.displayName = tmpversion+"-Dev-#"+currentBuild.number
			      		    				      
						}      
		      	  
				}}
			stage('Code Checkout') {
				steps {
				checkout scm
				}
			}	   
			stage('Munit Test') {
				steps {
					script {
						configFileProvider([configFile(fileId: '4042a019-5bca-4d87-a2b2-9f128dbb1015', variable: 'Maven_Settings')]) {
							echo "Munit Execution Begins"
							sh "mvn -s $Maven_Settings -Druntime.key=123 -DbuildNumber=${env.BUILD_NUMBER} -Denv=dev clean test" 
							echo "Munit completed"
							archive "target/munit-reports/coverage/**/*"
					   } 
				   }
			   }
			}
		    
		     stage('SonarQube Analysis') {
				steps {
					script {
				  withSonarQubeEnv('Sonarqube') {
				      configFileProvider([configFile(fileId: '4042a019-5bca-4d87-a2b2-9f128dbb1015', variable: 'Maven_Settings')]) {
				      sh "mvn -s $Maven_Settings sonar:sonar -Dsonar.projectKey=${pomArtifactId} -Dsonar.sources=$WORKSPACE/"
				      echo "sonar analysis completed"
                                  }
				     }
			               }}
				}	
                    stage('Upload To Nexus') {
				steps {
					script {
						configFileProvider([configFile(fileId: '4042a019-5bca-4d87-a2b2-9f128dbb1015', variable: 'Maven_Settings')]) {
							echo "Build project/Uploading to nexus Begins"
							sh "mvn -s $Maven_Settings clean package deploy -P nexus -DbuildNumber=${env.BUILD_NUMBER} -Denv=dev -DskipMunitTests" 
							echo "Build project and Uploading to nexus completed"
					   } 
				   }
			   }
			}	
		    
		    stage('deploy to exchange'){
		     steps {
			script{
				configFileProvider([configFile(fileId: '4042a019-5bca-4d87-a2b2-9f128dbb1015', variable: 'Maven_Settings')]) {
				withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
				sh "echo ${DEVOPSUSERNAME}"
				sh "mvn -s $Maven_Settings deploy -DbuildNumber=${env.BUILD_NUMBER} -DskipMunitTests"
				echo "exchange completed"
			      }}}}}
		    
		    
						
		    stage('deploy to rtf'){
			    steps {
			script{
				configFileProvider([configFile(fileId: '4042a019-5bca-4d87-a2b2-9f128dbb1015', variable: 'Maven_Settings')]) {
				withCredentials([usernamePassword(credentialsId: 'anypoint-platform', usernameVariable: 'DEVOPSUSERNAME', passwordVariable: 'DEVOPSPASSWORD')]) {
				sh "echo ${DEVOPSUSERNAME}"
				sh "mvn -s $Maven_Settings deploy -DmuleDeploy -DbuildNumber=${env.BUILD_NUMBER} -DskipMunitTests -Drtf.muleVersion=${RTF_MULE_VERSION_4_3_0} -Drtf.applicationName=${pomArtifactId} -DAnypoint.uri=${DEVOPS_MULE_ANYPOINT_URI} -Drtf.businessGroupId=${MULESOFT_ORG_ID} -Drtf.connectedAppClientId=$DEVOPSUSERNAME -Drtf.connectedAppClientSecret=$DEVOPSPASSWORD -Drtf.connectedAppGrantType=${DEVOPS_RTF_CONNECTEDAPPGRANTTYPE} -Drtf.environment=${DEVOPS_RTF_ENVIRONMENT}"
				}}}}}
				}
	
	  }