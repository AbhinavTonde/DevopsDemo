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
        git branch: "main", url: '<GITHUB REPOSITORY>'
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
          sh """
            ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '
              mkdir -p ${BACKUP_DIR} &&
              rm -rf ${BACKUP_DIR}/* &&
              if [ -d ${DEPLOY_DIR} ] && [ "$(ls -A ${DEPLOY_DIR})" ]; then
                cp -r ${DEPLOY_DIR}/* ${BACKUP_DIR}/
              else
                echo "No files to back up in ${DEPLOY_DIR}. Skipping backup."
              fi
            '
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ["devops"]) {
          sh """
            scp -r ./dist/my-dream-app/browser/* ${SSH_USER}@${SSH_HOST}:${DEPLOY_DIR}/
          """
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
        sh """
          ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '
            if [ -d ${BACKUP_DIR} ] && [ "$(ls -A ${BACKUP_DIR})" ]; then
              mkdir -p ${DEPLOY_DIR} &&
              rm -rf ${DEPLOY_DIR}/* &&
              cp -r ${BACKUP_DIR}/* ${DEPLOY_DIR}/
            else
              echo "No backup found. Rollback skipped."
            fi
          '
        """
      }

      error "Deployment failed and rollback attempted."
    }
  }
}
