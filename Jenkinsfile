

 

 node ('Ubuntu-app-agent'){
 
       
	   def app

   try {

     notifyBuild('STARTED')

					
					
		stage('Cloning Git') {
        /* Let's make sure we have the repository cloned to our workspace */
       checkout scm
    }

    stage('SAST') {
        build 'SCA-SAST-SNYK' 
    }
    
    stage('Build-and-Tag') {
    /* This builds the actual image; synonymous to
         * docker build on the command line */
        app = docker.build("afrzlfa/snake")
    }
    stage('Post-to-dockerhub') {
        docker.withRegistry('https://registry.hub.docker.com', 'training_creds') {
             app.push("latest")
        }
    }
    /* Test */
    
    stage('Pull-image-server') {
    
         sh "docker-compose down"
         sh "docker-compose up -d"	
      }
    
    stage('DAST') {
        build 'SECURITY-DAST-Arachni'
    }
					

     /* ... existing build steps ... */

 

   } catch (e) {

     // If there was an exception thrown, the build failed

     currentBuild.result = "FAILED"

     throw e

   } finally {

     // Success or failure, always send notifications

     notifyBuild(currentBuild.result)

   }

 }

 

 def notifyBuild(String buildStatus = 'STARTED') {

   // build status of null means successful

   buildStatus =  buildStatus ?: 'SUCCESSFUL'

 

   // Default values

   def colorName = 'RED'

   def colorCode = '#FF0000'

   def summary = "${subject} (${env.BUILD_URL})"
 

// Override default values based on build status



   if (buildStatus == 'STARTED') {



     color = 'YELLOW'



     colorCode = '#FFFF00'



   } else if (buildStatus == 'SUCCESSFUL') {



     color = 'GREEN'



     colorCode = '#00FF00'



   } else {



     color = 'RED'



     colorCode = '#FF0000'



   }
 

   // Send notifications

   slackSend (color: colorCode, message: summary)
 

 }

