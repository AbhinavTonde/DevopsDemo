pipeline {
  agent any

  environment {
    SSH_USER = 'ubuntu'
    DEPLOY_DIR = "/var/www/myapp"
    BACKUP_DIR = "/var/www/myapp_backup"
    SSH_HOST = "51.21.171.65" // Replace with actual IP
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "main", url: 'https://github.com/AbhinavTonde/DevopsDemo.git'
      }
    }

    stage('Build') {
      steps {
        echo "Installing npm"
        sh 'npm install'
        echo "Building the project"
        sh 'ng build'
      }
    }

    stage('Backup Current Version') {
      steps {
        sshagent (credentials: ["devops"]) {
          sh '''
            ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '
              BACKUP_DIR="/var/www/myapp_backup"
              DEPLOY_DIR="/var/www/myapp"

              mkdir -p "$BACKUP_DIR" &&
              if [ -d "$DEPLOY_DIR" ] && [ "$(ls -A "$DEPLOY_DIR")" ]; then
                rm -rf "$BACKUP_DIR"/* &&
                cp -r "$DEPLOY_DIR"/* "$BACKUP_DIR"/
              else
                echo "No files to back up in $DEPLOY_DIR. Skipping backup."
              fi
            '
          '''

        }
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ["devops"]) {
          sh '''
            scp -r ./dist/my-dream-app/browser/* ${SSH_USER}@${SSH_HOST}:${DEPLOY_DIR}/
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Deployment succeeded."
    }

    failure {
      echo "Deployment failed. Rolling back..."

      sshagent (credentials: ["devops"]) {
        sh '''
          ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '
            BACKUP_DIR="/var/www/myapp_backup"
            DEPLOY_DIR="/var/www/myapp"

            mkdir -p "$BACKUP_DIR" &&
            if [ -d "$DEPLOY_DIR" ] && [ "$(ls -A "$DEPLOY_DIR")" ]; then
              rm -rf "$BACKUP_DIR"/* &&
              cp -r "$DEPLOY_DIR"/* "$BACKUP_DIR"/
            else
              echo "No files to back up in $DEPLOY_DIR. Skipping backup."
            fi
          '
        '''

      }

      error "Deployment failed and rollback attempted."
    }
  }
}
