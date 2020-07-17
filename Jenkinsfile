
  
properties([parameters([choice(choices: 'master\nfeature-1\nfeature-2', description: 'Select the branch to build', name: 'branch')])])

node ('master'){
  //SCM Checkout
  stage('SCM Checkout'){
		try{
				git url: 'https://github.com/prasadkadam36/javahomecloud.git', branch: "${params.branch}"
		   }
    
		catch (errorCaught)
			{
				notify()
			}
  }
	
   stage('Compile-Package'){
		try{
				def mvnHome = tool name: 'apache-maven-3.5.4', type: 'maven'
				sh "${mvnHome}/bin/mvn package"
			}
		catch(errorCaught)
			{
				notify()
			}
   }
   
   
   stage('Code-Analysis'){
		try{
				def mvnHome = tool name: 'apache-maven-3.5.4', type: 'maven'
				withSonarQubeEnv('Sonar-7')
					{
						sh "${mvnHome}/bin/mvn sonar:sonar"
					}
			}
		catch(errorCaught)
			{
				notify()
			}
	}
   
   
  stage("Quality Gate"){
    
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
                notify()
              }
          }
      } 

	  
  stage('Push to Nexus'){
		try{
				def mvnHome = tool name: 'apache-maven-3.5.4', type: 'maven'
				sh "${mvnHome}/bin/mvn deploy"
			}
			catch (errorCaught)
			{
				notify()
			}
   }
  
 
   stage('Deploy to Tomcat'){
     
		try{
					   sh label: '', script: '''cd /var/lib/jenkins/workspace/PipelineasCode
				VERSION=`cat pom.xml | grep version -m1 | awk -F ">" \'{print $2}\' | awk -F "<" \'{print $1}\'`
				cd /home/ansadmin/
				rm -rf *.war
				wget http://34.68.222.117:8081/repository/maven-releases/in/javahome/myweb/"${VERSION}"/myweb-"${VERSION}".war
				mv myweb-"${VERSION}".war myweb.war
				scp -rp *.war ansadmin@10.128.0.22:/home/ansadmin
				'''
			}
		catch (errorCaught)
			{
       notify()
			}
   
  
   stage('Email Notification'){
	
	 emailext body: '${currentBuild.currentResult} : Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \\n More info at : ${env.BUILD_URL}', subject: 'Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}', to: 'ankitsoni.570@gmail.com'
	
    }



notify()
	{
	 emailext body: '${currentBuild.currentResult} : Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \\n More info at : ${env.BUILD_URL}', subject: 'Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}', to: 'ankitsoni.570@gmail.com'
	}
   }
}
