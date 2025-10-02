pipeline {
  agent any
  options { skipStagesAfterUnstable() }
  environment {
    APP_NAME = 'sjsu-ansible'
    ANSIBLE_FORCE_COLOR = '1'
    // Leave credentials commented unless you really created one with that ID:
    // API_TOKEN = credentials('example-api-token')
  }

  stages {
    stage('Hello') { steps { echo 'Hello from Jenkins' } }

    stage('Checkout') { steps { checkout scm } }

    stage('Setup (Python & Ansible)') {
      steps {
        sh '''set -e
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
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          sh '''set -e
. .venv/bin/activate
ansible-lint -v site.yml | tee logs/ansible-lint.txt
'''
        }
      }
    }

    stage('Syntax check') {
      steps {
        sh '''set -e
. .venv/bin/activate
ansible-playbook -i inventory.ini site.yml --syntax-check | tee logs/syntax-check.txt
'''
      }
    }

    stage('Dry run (check mode)') {
      steps {
        sh '''set -e
. .venv/bin/activate
ansible-playbook -i inventory.ini site.yml --check --diff | tee logs/check-mode.txt
tar -czf dist/${APP_NAME}-${BUILD_NUMBER}.tgz site.yml inventory.ini group_vars host_vars templates || true
'''
      }
    }

    stage('Test report (demo)') {
      steps {
        writeFile file: 'reports/junit.xml', text: '<testsuite name="ansible-demo
