node('Ubuntu-app-agent') {
    def app

    try {
        notifyBuild('STARTED')

        stage('Cloning Git') {
            // Ensure the repository is cloned to the workspace
            checkout scm
        }

        stage('SAST') {
            // Run Static Application Security Testing (SAST)
            build 'SCA-SAST-SNYK'
        }

        stage('Build-and-Tag') {
            // Build the Docker image
            app = docker.build("afrzlfa/snake")
        }

        stage('Post-to-dockerhub') {
            // Push the image to DockerHub
            docker.withRegistry('https://registry.hub.docker.com', 'training_creds') {
                app.push("latest")
            }
        }

        stage('Pull-image-server') {
            // Deploy the Docker image using docker-compose
            sh "docker-compose down"
            sh "docker-compose up -d"
        }

        stage('DAST') {
            // Run Dynamic Application Security Testing (DAST)
            build 'SECURITY-DAST-Arachni'
        }

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
    // Build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESSFUL'

    // Default values
    def color = 'RED'
    def colorCode = '#FF0000'
    def summary = "${env.JOB_NAME} - ${buildStatus} (${env.BUILD_URL})"

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#00FF00'
    }

    // Send notifications
    slackSend(color: colorCode, message: summary)
}
