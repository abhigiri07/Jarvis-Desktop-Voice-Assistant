pipeline {
  agent any

  environment {
    TARGET = "ubuntu@3.94.162.152"
  }

  triggers {
    githubPush()
  }

  stages {

    stage("Checkout") {
      steps {
           git url: 'https://github.com/abhigiri07/Jarvis-Desktop-Voice-Assistant.git', branch: 'main'
      }
    }

    stage("Deploy to EC2") {
      steps {
        sshagent(['new-key']) {
          sh '''
            echo ">>> Connecting to EC2..."
            ssh -o StrictHostKeyChecking=no ubuntu@3.94.162.152 "echo Connected"

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
