def dockerRepo = "ghcr.io/cancogen-virus-seq/keycloak"
def gitHubRepo = "cancogen-virus-seq/cancogen-keycloak-theme"
def commit = "UNKNOWN"

pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind-daemon
    image: docker:18.06-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ''
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  - name: helm
    image: alpine/helm:2.12.3
    command:
    - cat
    tty: true
  - name: docker
    image: docker:18-git
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
    - name: HOME
      value: /home/jenkins/agent
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      fsGroup: 1000
  volumes:
  - name: dind-storage
    emptyDir: {}

"""
        }
    }
    stages {
        stage('Build & Publish') {
            when {
                branch "main"
            }
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'argoContainers', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login ghcr.io -u $USERNAME -p $PASSWORD'
                    }

                    // DNS error if --network is default
                    sh "docker build --network=host . -t ${dockerRepo}:latest -t ${dockerRepo}:${commit}"

                    sh "docker push ${dockerRepo}:$${commit}"
                    sh "docker push ${dockerRepo}:latest"
                }
            }
        }
}
