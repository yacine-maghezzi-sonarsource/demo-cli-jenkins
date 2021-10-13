buildDir = "build"
pylintReport = "${buildDir}/pylint-report.out"
banditReport = "${buildDir}/bandit-report.json"
flake8Report = "${buildDir}/flake8-report.out"
coverageReport = "${buildDir}/coverage.xml"

coverageTool = 'coverage'
pylintTool = 'pylint'
flake8Tool = 'flake8'
banditTool = 'bandit'

// rm -rf -- ${buildDir:?"."}/* .coverage */__pycache__ */*.pyc # mediatools/__pycache__  testpytest/__pycache__ testunittest/__pycache__

pipeline {
  agent any
  stages {
    stage('Code Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '**']], extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false]], userRemoteConfigs: [[credentialsId: 'github-app', url: 'https://github.com/okorach/demo-maven-jenkins']]])
      }
    }
    stage('Run tests') {
      steps {
        script {
          echo "Run unit tests for coverage"
          sh "cd comp-cli; ${coverageTool} run -m pytest"
          echo "Generate XML report"
          sh "pwd; cd comp-cli; ${coverageTool} xml -o ${coverageReport}"
        }
      }
    }
    stage('Run 3rd party linters') {
     steps {
        script {
          // sh "cd comp-cli; ${pylintTool} *.py */*.py -r n --msg-template=\"{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}\" > ${pylintReport}"
          // sh "cd comp-cli; ${flake8Tool} --ignore=W503,E128,C901,W504,E302,E265,E741,W291,W293,W391 --max-line-length=150 . > ${flake8Report}"
          // sh "cd comp-cli; ${banditTool} -f json --skip B311,B303 -r . -x .vscode,./testpytest,./testunittest > ${banditReport}"
          sh "cd comp-cli; ./run_linters.sh"
        }
      }
    }
    stage('SonarQube LATEST analysis') {
      steps {
        withSonarQubeEnv('SQ Latest') {
          script {
            def scannerHome = tool 'SonarScanner';
            sh "cd comp-cli; ${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
    stage("SonarQube LATEST Quality Gate") {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              echo "Quality gate failed: ${qg.status}, proceeding anyway"
            }
          }
        }
      }
    }
  }
}