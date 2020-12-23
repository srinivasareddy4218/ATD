node{
     
    def mvnHome = tool 'maven-3.3.9'
         // holds reference to docker image
    def dockerImage
    // ip address of the docker private repository(nexus)
   
	
    stage('SCM Checkout'){
        git credentialsId: 'aabc9fb7-0647-4c60-93ce-e92eeabb6252', url: 'https://github.com/oohasri95/Sample1.git'
    }
 
  
    stage('Build Project') {
      // build project via maven
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
    }
	
    stage('Publish Tests Results'){
      parallel(
        publishJunitTestsResultsToJenkins: {
          echo "Publish junit Tests Results"
		  junit '**/target/surefire-reports/TEST-*.xml'
		  archive 'target/*.jar'
        },
        publishJunitTestsResultsToSonar: {
          echo "This is branch b"
      })
    }
	
    //stage('Build Docker Image') {
      // build docker image
    //  sh "whoami"
    //  sh "ls -all /var/run/docker.sock"
    //  sh "mv ./target/hello*.jar ./data" 
      
    //  dockerImage = docker.build("hello-world-java")
	    
     stage('Build Docker Image'){
	    sh "sudo docker build -t us.gcr.io/mssdevops-284216/sample-java ."
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
        sh "sudo docker push us.gcr.io/mssdevops-284216/sample-java" 
        }
   }
	 
     stage('Create Cluster GKE') {
	withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
	sh "gcloud config set project ${projectname}"
        sh "gcloud config set compute/zone ${zone}"
        sh "gcloud config set compute/region ${region}"
        sh "gcloud auth configure-docker"
        sh "gcloud config list"
	sh "gcloud container clusters create sample6-cluster \
--machine-type=e2-medium"
   }
   
     }	     
	stage('Deploy to kubernetes'){
        withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
	sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
//Configuring the project details to Jenkins and communicate with the gke cluster
         sh "gcloud config set project ${projectname}"
         sh "gcloud config set compute/zone ${zone}"
         sh "gcloud config set compute/region ${region}"
         sh "gcloud container clusters get-credentials sample6-cluster --zone us-central1-a --project mssdevops-284216"
	 sh "kubectl create namespace samplejava"
	 sh "kubectl apply -f sampledeploy.yml"
 }
			} 
	
	stage('Destroy Cluster'){
        withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
	sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
//Configuring the project details to Jenkins and communicate with the gke cluster
         sh "gcloud config set project ${projectname}"
         sh "gcloud config set compute/zone ${zone}"
         sh "gcloud config set compute/region ${region}"
         sh "gcloud container clusters get-credentials sample6-cluster --zone us-central1-a --project mssdevops-284216"
	 sh "gcloud container -q clusters delete sample6-cluster --zone=us-central1-a"
	}
	}

}
