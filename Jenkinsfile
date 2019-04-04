def label = "jenkins-input-repro-${UUID.randomUUID().toString()}"
def scmVars;


def installDeps() {
  stage('Install dependencies') {
    container('ci') {
      scmVars = checkout scm
      sh 'sleep 10'
    }
  }
}

def build(target) {
  this.installDeps()

  stage('Build and push image') {
    container('ci') {
      sh 'sleep 10'
    }
  }
}

def deploy(target) {
  stage("Deploy to ${target}") {
    container('ci') {
    }
  }
}

podTemplate(label: label, podRetention: onFailure(), activeDeadlineSeconds: 600, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: ci
      image: go:latest
      tty: true
      resources:
        requests:
          memory: "4Gi"
          cpu: "2"
        limits:
          memory: "6Gi"
          cpu: "3"
"""
  ) {
  node(label) {
    this.installDeps()

    stage('Plan and Test') { 
      container('ci') {
        sh 'sleep 10'
      }
    }
  }

  if ("${env.BRANCH_NAME}" != "master") {
    stage('Approve dev') {
      input message: 'Deploy to dev?'
    }

    node(label) {
      try {
        this.build('dev')
        this.deploy('dev')
      } catch (err) {
        throw err;
      }
    }
  }

  if ("${env.BRANCH_NAME}" == "master") {
    stage('Approve staging') {
      input message: 'Deploy to staging?'
    }

    node(label) {
      try {
        this.build('staging')
        this.deploy('staging')
      } catch (err) {
        throw err;
      }
    }

    stage('Approve prod') {
      input message: 'Deploy to prod?'
    }

    node(label) {
      try {
        this.deploy('prod')
      } catch (err) {
        throw err;
      }
    }
  }
}