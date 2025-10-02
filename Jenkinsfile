pipeline {
  agent any
  options { skipStagesAfterUnstable() }
  environment {
    APP_NAME = 'sjsu-ansible'
    ANSIBLE_FORCE_COLOR = '1'
    // Leave this commented unless you actually created a credential with that ID:
    // API_TOKEN = credentials('example-api-token')
  }

  stages {
    stage('Hello') {
      steps { echo 'Hello from Jenkins' }
    }

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Setup (Python & Ansible)') {
      steps {
        sh 'python3 -V'
        sh 'python3 -m venv .venv'
        sh """. .venv/bin/activate
pip install --upgrade pip
pip install "ansible<11" ansible-lint
ansible --version
ansible-lint --version
mkdir -p reports logs dist"""
      }
    }

    stage('Lint (ansible-lint)') {
      steps {
        catchError(buildResult: "UNSTABLE", stageResult: "FAILURE") {
          sh """. .venv/bin/activate
ansible-lint -v site.yml | tee logs/ansible-lint.txt"""
        }
      }
    }

    stage('Syntax check') {
      steps {
        sh """. .venv/bin/activate
ansible-playbook -i inventory.ini site.yml --syntax-check | tee logs/syntax-check.txt"""
      }
    }

    stage('Dry run (check mode)') {
      steps {
        sh """. .venv/bin/activate
ansible-playbook -i inventory.ini site.yml --check --diff | tee logs/check-mode.txt"""
        sh "tar -czf dist/${APP_NAME}-${BUILD_NUMBER}.tgz site.yml inventory.ini group_vars host_vars templates || true"
      }
    }

    stage('Test report (demo)') {
      steps {
        writeFile file: 'reports/junit.xml',
                 text: '<testsuite name="ansible-demo" tests="1"><testcase classname="demo" name="pass"/></testsuite>'
      }
    }

    stage('Deploy - Staging') {
      when { branch 'main' }
      steps {
        echo 'Pretend deploy to STAGING…'
        sh '. .venv/bin/activate && echo "Staging deploy step completed"'
      }
    }

    stage('Approval to Production') {
      when { branch 'main' }
      steps {
        input id: 'prod-approval',
              message: "Promote build ${env.BUILD_NUMBER} to PRODUCTION?",
              ok: 'Deploy'
      }
    }

    stage('Deploy - Production') {
      when { branch 'main' }
      steps {
        echo 'Pretend deploy to PRODUCTION…'
        sh '. .venv/bin/activate && echo "Production deploy step completed"'
      }
    }

  }

  post {
    always {
      junit 'reports/**/*.xml'
      archiveArtifacts artifacts: 'dist/**/*, logs/**/*', fingerprint: true
      echo "Build URL: ${env.BUILD_URL}"
      deleteDir()
    }
  }
}
