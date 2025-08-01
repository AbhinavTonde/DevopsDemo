pipeline {
  agent any

  environment {
    DEPLOY_DIR = "/var/www/myapp"
    BACKUP_DIR = "/var/www/myapp_backup"
    BUILD_DIR = "dist/my-dream-app/browser"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "main", url: 'https://github.com/AbhinavTonde/DevopsDemo.git'
      }
    }

    stage('Build') {
      steps {
        sh 'npm install'
        sh 'npx ng build --configuration production'
      }
    }

    stage('Backup Current Version') {
      steps {
        script {
          sh '''
            mkdir -p "${BACKUP_DIR}"
            if [ -d "${DEPLOY_DIR}" ] && [ "$(ls -A ${DEPLOY_DIR})" ]; then
              rm -rf "${BACKUP_DIR}"/*
              cp -r "${DEPLOY_DIR}"/* "${BACKUP_DIR}/"
            else
              echo "Nothing to backup in ${DEPLOY_DIR}"
            fi
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh '''
            mkdir -p "${DEPLOY_DIR}"
            rm -rf "${DEPLOY_DIR}"/*
            cp -r "${BUILD_DIR}/"* "${DEPLOY_DIR}/"
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful!"
    }
    failure {
      echo "❌ Deployment failed. Rolling back..."
      script {
        sh '''
          if [ -d "${BACKUP_DIR}" ] && [ "$(ls -A ${BACKUP_DIR})" ]; then
            rm -rf "${DEPLOY_DIR}"/*
            cp -r "${BACKUP_DIR}/"* "${DEPLOY_DIR}/"
            echo "Rollback complete."
          else
            echo "Nothing to rollback. Backup not found or empty."
          fi
        '''
      }
    }
  }
}
