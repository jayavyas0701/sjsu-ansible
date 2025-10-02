pipeline {
  agent any
  options { skipStagesAfterUnstable() }
  environment {
    APP_NAME = 'sjsu-ansible'
    ANSIBLE_FORCE_COLOR = '1'
    // If you actually created a Jenkins credential with this ID, uncomment next line.
    // API_TOKEN = credentials('example-api-token')
  }

  stages {
    stage('Hello') {
      steps { echo 'Jenkins sees this Jenkinsfile ✅' }
    }

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Setup (Python & Ansible)') {
      steps {
        sh '''
          set -e
          python3 -V
          python3 -m venv .venv
          . .venv/bin/activate
          pip install --upgrade pip
          pip install "ansible<11" ansible-lint
          ansible --version
          ansible-lint --version
          mkdir -p reports logs dist
        '''
      }
    }

    stage('Lint (ansible-lint)') {
      steps {
