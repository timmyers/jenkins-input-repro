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

def runInPod(thing) {
  def label = "jenkins-input-repro-${UUID.randomUUID().toString()}"

  podTemplate(label: label, podRetention: onFailure(), activeDeadlineSeconds: 600, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: ci
      image: golang:latest
      tty: true
"""
  ) { 
    node(label) {
      thing
    }
  }
}

runInPod({
  this.deploy('staging');
  this.endToEndTests('staging')
})

stage('Approve prod') {
  input message: 'Deploy to prod?'
}


runInPod({
  this.deploy('prod')
  this.endToEndTests('prod')
})
