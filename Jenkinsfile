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
            url: 'https://github.com/abhigiri07/Jarvis-Desktop-Voice-Assistant-Project.git',
            branch: 'main'
      }
    }

    stage("Deploy to EC2") {
      steps {
        sshagent(['jenkins-key']) {
          sh '''
          ssh -o StrictHostKeyChecking=no ubuntu@52.91.244.151 "echo Connected"
          rsync -avz --delete --exclude venv . ${TARGET}:/opt/jarvis
          ssh ${TARGET} "sudo systemctl restart jarvis"
          '''
        }
      }
    }
  }
}
