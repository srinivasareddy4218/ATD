node{
     
    def mvnHome = tool 'maven-3.3.9'
         // holds reference to docker image
    def dockerImage
    // ip address of the docker private repository(nexus)
   
	
    //stage('SCM Checkout'){
        //git credentialsId: 'aabc9fb7-0647-4c60-93ce-e92eeabb6252', url: 'https://github.com/Lohita20/ATD.git'
    //}
 
    stage('Build Project'){
      parallel(
        Project1: {
          cd sample
	  sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package" 
	},
		
	Project2: {
	  cd test
	  sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package" 
	})
    }

    //stage ('Publish Test Results'){
      //parallel(
        //Project1: {
          //parallel(
	    //publishJunitTestsResultsToJenkins: {
              //echo "Publish junit Tests Results"
	      //cd sample
	      //junit '**/target/surefire-reports/TEST-*.xml'
	      //archive 'target/*.jar'
            //},
            //publishJunitTestsResultsToSonar: {
              //echo "This is branch test"
            //})
	//},
		
	//Project2: {
          //parallel(
	    //publishJunitTestsResultsToJenkins: {
              //echo "Publish junit Tests Results"
	      //cd test
	      //junit '**/target/surefire-reports/TEST-*.xml'
	      //archive 'target/*.jar'
            //},
            //publishJunitTestsResultsToSonar: {
              echo "This is branch sample"
            //})
        //}
      //)
    //}
    	
    stage('Build Docker Image'){
      parallel(
        BuildDockerImageForProject1: {
	  cd sample
          sh "sudo docker build -t us.gcr.io/mssdevops-284216/sample-java1 ."
	},	
        
	BuildDockerImageForProject2: {
          cd test
	  sh "sudo docker build -t us.gcr.io/mssdevops-284216/sample-java2 ."		
	})
    }
    
    stage('GCR packaging') {
        withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
        sh "gcloud config set project ${projectname}"
        sh "gcloud config set compute/zone ${zone}"
        sh "gcloud config set compute/region ${region}"
        sh "gcloud auth configure-docker"
        sh "gcloud config list"
        sh "cat ${GOOGLE_APPLICATION_CREDENTIALS} | sudo docker login -u _json_key --password-stdin https://us.gcr.io"
        sh "sudo docker push us.gcr.io/mssdevops-284216/sample-java1" 
        sh "sudo docker push us.gcr.io/mssdevops-284216/sample-java2" 

        }
    }
}
