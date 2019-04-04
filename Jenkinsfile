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

def endToEndTests(target) {
  container('ci') {
    stage('End to end tests') {
      parallel '1': {
        sh 'sleep 10'
      }, '2': {
        sh 'sleep 10'
      }, '3': {
        sh 'sleep 10'
      }
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
metadata:
  annotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
spec:
  containers:
    - name: ci
      image: 204717343847.dkr.ecr.us-east-1.amazonaws.com/infura/infrastructure-ci-go:latest
      tty: true
      volumeMounts:
      - name: dockersock
        mountPath: /var/run/docker.sock
      resources:
        requests:
          memory: "4Gi"
          cpu: "2"
        limits:
          memory: "6Gi"
          cpu: "3"

  volumes:
    - name: dockersock
      hostPath:
        path: /var/run/docker.sock
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
        this.endToEndTests('dev')
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
        this.endToEndTests('staging')
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
        this.endToEndTests('prod')
      } catch (err) {
        throw err;
      }
    }
  }
}
