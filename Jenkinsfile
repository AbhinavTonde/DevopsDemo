pipeline {
  agent any

  parameters {
    string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0', description: 'Deployment version')
    string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to deploy')
    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
  }

  environment {
    SSH_USER = 'ubuntu'
    DEPLOY_DIR = "/var/www/myapp"
    BACKUP_DIR = "/var/www/myapp_backup"
    SSH_HOST = "51.21.191.169"
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
        echo "Installing angular"
        sh 'npm install -g @angular/cli'
        echo "Building the project"
        sh 'ng build'
        echo "Serving the project"
        sh 'ng build --watch --configuration development'
      }
    }

    stage('Backup Current Version') {
      steps {
        sshagent (credentials: ["server-dev"]) {
          sh """
            ssh ${SSH_USER}@${SSH_HOST} '
              mkdir -p ${BACKUP_DIR} &&
              rm -rf ${BACKUP_DIR}/* &&
              cp -r ${DEPLOY_DIR}/* ${BACKUP_DIR}/
            '
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ["server-dev"]) {
          sh """
            scp -r ./dist/my-dream-app/browser
          """
        }
      }
    }
  }

  post {
    success {
      echo "Deployment ${params.DEPLOY_VERSION} to ${params.ENV} succeeded."
    }

    failure {
      echo "Deployment failed. Rolling back..."

      sshagent (credentials: ["server-dev"]) {
        sh """
          ssh ${SSH_USER}@${SSH_HOST} '
            mkdir -p ${BACKUP_DIR} &&
            rm -rf ${DEPLOY_DIR}/* &&
            cp -r ${BACKUP_DIR}/* ${DEPLOY_DIR}/
          '
        """
      }

      error "Deployment failed and rollback completed."
    }
  }
}
