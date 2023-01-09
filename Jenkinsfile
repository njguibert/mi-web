pipeline {
  environment { 
    APP_VERSION = '0.0.1'
    IMG_NAME = 'miweb'
    DOCKER_REGISTRY = 'https://gcr.io'
    DOCKER_REPO = 'njguibert'
    DOCKER_IMG_NAMEREPO = 'mi-web'
    GIT_REPO = 'njguibert/mi-web'
    ARGOCD_SERVER='argocd-server.argocd'
    ARGOCD_PROJECT='jesusguibert.com'
    ARGOCD_APITOKEN='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJtaWNyb2s4czphcGlLZXkiLCJuYmYiOjE2NzMwOTg4NjYsImlhdCI6MTY3MzA5ODg2NiwianRpIjoiZTA4YzRjNWYtZjI2Yy00YTBhLWFkYjAtY2QxNTU0MTZkMDNiIn0.9JKY6Cy99AQ_JoCeuwznJk1djXkutmKpnFpAII5dUoA'
    NAMESPACE='argotest'
  }    
  agent {
    kubernetes {
      yaml """
              kind: Pod
              spec:
                containers:
                - name: cli-tools
                  image: argoproj/argo-cd-ci-builder
                  command:
                    - cat
                  tty: true                                      
                - name: kaniko
                  image: gcr.io/kaniko-project/executor:debug
                  imagePullPolicy: Always
                  command:
                  - sleep
                  args:
                  - 99d
                  volumeMounts:
                    - name: jenkins-docker-cfg
                      mountPath: /kaniko/.docker
                volumes:
                - name: jenkins-docker-cfg
                  projected:
                    sources:
                    - secret:
                        name: regcred
                        items:
                          - key: .dockerconfigjson
                            path: config.json
"""
    }
  }
  stages {
    stage('Build Dockerimage with kaniko') {
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
        ///kaniko/executor -f `pwd`/Dockerfile --context 'git://github.com/njguibert/mi-web' --destination njguibert/mi-web:latest
          sh '''#!/busybox/sh
            /kaniko/executor --context "git://github.com/${GIT_REPO}" --destination ${DOCKER_REPO}/${DOCKER_IMG_NAMEREPO}:${APP_VERSION}-${BUILD_NUMBER}
          '''
        }
      }
    }
   
    stage('Update argocd manifest') {
      steps {
        container(name: 'cli-tools') {
            withCredentials([gitUsernamePassword(credentialsId: 'github_njguibert', gitToolName: 'git-tool')]) {
              sh "git clone https://github.com/njguibert/homelab-manifest.git"
              sh "git config --global user.email 'ci@ci.com'"          
              dir("homelab-manifest") {
                sh "pwd"
                sh "ls"
                sh "cd  prod/argocd/jesusguibert.com && sed -i -E \"/- name: nginx/{n;s~(image: ).*~\\1$DOCKER_REPO/$DOCKER_IMG_NAMEREPO:$APP_VERSION-${BUILD_NUMBER}~}\" 01-deployment.yaml"
                sh "git commit -am 'Tag actualizada' && git push "
              }              
            }
        }
      }
    }    
    stage('Push to k8s') {
      steps {
          container(name: 'cli-tools') {
            sh "echo Publicando a K8s...."
            sh "wget -O argocd http://${ARGOCD_SERVER}/download/argocd-linux-amd64 "
            //sh "curl -sSL -k -o argocd https://${ARGOCD_SERVER}/download/argocd-linux-amd64"
            sh "chmod 755 argocd"
            //sh "./argocd app sync ${ARGOCD_PROJECT} --auth-token ${ARGOCD_APITOKEN} --plaintext"
            sh "./argocd app sync ${ARGOCD_PROJECT} --auth-token ${ARGOCD_APITOKEN} --plaintext"
            sh "./argocd app wait -l app.kubernetes.io/instance=${ARGOCD_PROJECT} --sync --health --timeout 60 --auth-token ${ARGOCD_APITOKEN} --plaintext"
          }
      }    
    }
           
  }
}
