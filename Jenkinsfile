def label = "jenkins-input-repro-${UUID.randomUUID().toString()}"
def scmVars;

def endToEndTests(target) {
  stage('End to end tests') {
    container('ci') {
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

def deploy(target) {
  stage("Deploy to ${target}") {
    container('ci') {
      sh 'sleep 10'
    }
  }
}

podTemplate(label: label, podRetention: onFailure(), activeDeadlineSeconds: 600, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: ci
      image: golang:latest
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
    this.deploy('staging')
    this.endToEndTests('staging')
  }

  stage('Approve prod') {
    input message: 'Deploy to prod?'
  }

  node(label) {
    this.deploy('prod')
    this.endToEndTests('prod')
  }
}
