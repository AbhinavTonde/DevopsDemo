pipeline {
  agent any

  parameters {
    string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0', description: 'Deployment version')
    string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to deploy')
    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
  }

  environment {
    SSH_USER = 'ubuntu' // change if needed
    DEPLOY_DIR = "/var/www/myapp" // adjust this
    BACKUP_DIR = "/var/www/myapp_backup"
    SSH_HOST = "16.171.154.7" // Use Jenkins credentials ID for SSH per environment
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "main", url: 'https://github.com/AbhinavTonde/DevopsDemo.git'
      }
    }

    stage('Build') {
      steps {
        echo "Checking npm version"
        sh 'npm -v'

        echo "Installing project dependencies"
        sh 'npm install'

        echo "Installing Angular CLI"
        sh 'npm install -g @angular/cli@18'

        echo "Building the Angular project"
        sh 'ng build --watch --configuration development'
    }

    stage('Backup Current Version') {
      steps {
        sshagent (credentials: ["server-${params.ENV}"]) {
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
        sshagent (credentials: ["server-${params.ENV}"]) {
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
