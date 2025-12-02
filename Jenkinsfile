pipeline {
  agent any

  environment {
    TARGET = "ubuntu@52.91.244.151"
  }

  triggers {
    githubPush()
  }

  stages {

    stage("Checkout") {
      steps {
        git credentialsId: 'github_pat',
            url: 'https://github.com/abhigiri07/Jarvis-Desktop-Voice-Assistant.git',
            branch: 'main'
      }
    }

    stage("Deploy to EC2") {
      steps {
        sshagent(['jenkins-key']) {
          sh '''
            echo ">>> Connecting to EC2..."
            ssh -o StrictHostKeyChecking=no ${TARGET} "echo Connected"

            echo ">>> Syncing project to /opt/jarvis..."
            rsync -avz --delete --exclude venv . ${TARGET}:/opt/jarvis

            echo ">>> Restarting service..."
            ssh ${TARGET} "sudo systemctl restart jarvis"

            echo ">>> Deployment complete!"
          '''
        }
      }
    }
  }
}
