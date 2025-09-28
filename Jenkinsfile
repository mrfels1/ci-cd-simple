pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }

  parameters {
    choice(name: 'DEPLOY_ENV', choices: ['dev','staging','prod'], description: 'Среда деплоя')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        bat 'mkdir dist'
        script {
          writeFile file: 'dist/app.txt',
                   text: "hello, build at ${new Date().format('yyyy-MM-dd HH:mm:ss')}\n"
        }
      }
    }

    stage('Test') {
      steps {
        bat 'mkdir reports'
        script {
          writeFile file: 'reports/junit-sim.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="simulated" tests="2" failures="0" errors="0" time="0.0">
  <testcase classname="sim.Build" name="AppFileExists" time="0.0"/>
  <testcase classname="sim.Smoke" name="AlwaysGreen" time="0.0"/>
</testsuite>
'''
        }
        junit testResults: 'reports/junit-*.xml'
      }
    }

    stage('Package') {
      steps {
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
      }
    }

    stage('Approve PROD') {
      when { expression { params.DEPLOY_ENV == 'prod' } }
      steps { input message: 'Подтвердить деплой в PROD?', ok: 'Продолжить' }
    }

    stage('Deploy') {
      steps {
        script {
          def msg = "deployed to ${params.DEPLOY_ENV} at ${new Date().format('yyyy-MM-dd HH:mm:ss')}\n"
          writeFile file: "reports/deploy-${params.DEPLOY_ENV}.log", text: msg
          echo msg
        }
      }
    }
  }

  post {
    always { archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true }
  }
}
