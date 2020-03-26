node {
	currentBuild.displayName = "1.${BUILD_NUMBER}"
	def GIT_COMMIT
  stage ('cloning the repository'){
	  
      def scm = git 'https://github.com/hclpowership/JPetstore'
	  GIT_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
	  echo "AAAA ${GIT_COMMIT}"
	  //echo "BBBB ${scm}"
	  //GIT_COMMIT = scm.GIT_COMMIT
	   //echo "**** ${GIT_COMMIT}"
	  
  }
	
 
  stage ('Build') {
      withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
      sh 'mvn clean package'
	      echo "**** ${GIT_COMMIT}"
	//step($class: 'UploadBuild', tenantId: "5ade13625558f2c6688d15ce", revision: "${GIT_COMMIT}", appName: "PROJECT_NAME", requestor: "admin", id: "${newComponentVersionId}" )
	
	     
    }
  }
  stage('JUnit'){
		echo("************************** Test Result Upload Started to Velocity****************************")
                        try{
                        step([$class: 'UploadJUnitTestResult',
                            properties: [
                        // Need to change the path of the test result xml result required.               
                                filePath: "target/surefire-reports/TEST-org.mybatis.jpetstore.service.OrderServiceTest.xml",
                                tenant_id: "5ade13625558f2c6688d15ce",
                                appName: "JPetStore",
                                //appExtId: "4b006cdb-0e50-43f2-ac87-a7586a65389e",    
                                name: "Executed in JUnit - 3.0.${BUILD_NUMBER}",
                                testSetName: "Junit Test Run from Jenkins"]
                           
                        ])}catch(e){
                        throw e
                        }
                       
         echo("************************** Test Result Uploaded Successful to Velocity****************************")

		
	}

	stage('SonarQube Analysis'){
		def mvnHome = tool name : 'MVN_Local', type:'maven'
		withSonarQubeEnv('sonar-server'){
			 "SONAR_USER_HOME=/opt/bitnami/jenkins/.sonar ${mvnHome}/bin/mvn sonar:sonar"
			sh  "${mvnHome}/bin/mvn sonar:sonar -Dsonar.projectKey=jpetstore -Dsonar.projectName=JPetStore"
		}
	}
	
	
stage ("Appscan"){
	//sleep 40
	// appscan application: '84963f4f-0cf4-4262-9afe-3bd7c0ec3942', credentials: 'Credential for ASOC', failBuild: true, failureConditions: [failure_condition(failureType: 'high', threshold: 100)], name: '84963f4f-0cf4-4262-9afe-3bd7c0ec39426367', scanner: static_analyzer(hasOptions: false, target: '/var/jenkins_home/jobs/jpetstore'), type: 'Static Analyzer'
 }
  stage('Publish Artificats to UCD'){
	  
   step([$class: 'UCDeployPublisher',
        siteName: 'ucd-server',
        component: [
            $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
            componentName: 'JPetStoreComponent',
            createComponent: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
                componentTemplate: '',
                componentApplication: 'JPetStore'
            ],
            delivery: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
                pushVersion: '1.${BUILD_NUMBER}',
                //baseDir: '/var/jenkins_home/workspace/JPetStore/target',
		    baseDir: '/var/jenkins_home/workspace/${JOB_NAME}/target',
                fileIncludePatterns: '*.war',
                fileExcludePatterns: '',
               // pushProperties: 'jenkins.server=Jenkins-app\njenkins.reviewed=false',
                pushDescription: 'Pushed from Jenkins'
            ]
        ]
    ])
	  
		//sh 'env > env.txt'
	//	readFile('env.txt').split("\r?\n").each {
	//	println it
	//	}
	echo "(*****)"
	  echo "${UUID}"
	
	  echo "Demo1234 ${JPetStoreComponent_VersionId}"
	  def newComponentVersionId = "${JPetStoreComponent_VersionId}"
	  step($class: 'UploadBuild', tenantId: "5ade13625558f2c6688d15ce", revision: "${GIT_COMMIT}", appName: "JPetStore", requestor: "admin", id: "${newComponentVersionId}" )
	  echo "Demo123 ${newComponentVersionId}"
	sleep 25
	  step([$class: 'UCDeployPublisher',
		deploy: [ createSnapshot: [deployWithSnapshot: true, 
			 snapshotName: "1.${BUILD_NUMBER}"],
			 deployApp: 'JPetStore', 
			 deployDesc: 'Requested from Jenkins', 
			 deployEnv: 'JPetStore_Dev', 
			 deployOnlyChanged: false, 
			 deployProc: 'Deploy', 
			 deployReqProps: '', 
			 deployVersions: "JPetStoreComponent:1.${BUILD_NUMBER}"], 
		siteName: 'ucd-server'])
 }

stage ("Appscan DAST"){
//	appscan application: '84963f4f-0cf4-4262-9afe-3bd7c0ec3942', credentials: 'Credential for ASOC', failBuild: true, failureConditions: [failure_condition(failureType: 'high', threshold: 100)], name: 'Dast', scanner: dynamic_analyzer(hasOptions: false, scanType: 'Staging', target: 'http://34.74.72.221:8080/jpetstore'), type: 'Dynamic Analyzer'
}
	
stage ('HCL OneTest') {
	// sleep 25
	echo 'Executing HCL One test ... '
	 sh '/var/jenkins_home/onetest/execute-workspace.sh jpetstore-demo http://35.231.119.193:8080'
 }

}

