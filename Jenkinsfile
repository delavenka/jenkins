pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  hostNetwork: true
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: 'IfNotPresent'
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-credentials
        items:
          - key: .dockerconfigjson
            path: config.json
"""
        }
    }

    environment {
        DOCKER_USER = "idilsuelmas"
        BACKEND_IMAGE = "idilsuelmas/todo-backend"
        FRONTEND_IMAGE = "idilsuelmas/todo-frontend"
    }

    stages {
        stage('Build & Push Backend') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context \${WORKSPACE} \
                    --dockerfile \${WORKSPACE}/todo-backend/Dockerfile \
                    --destination \${BACKEND_IMAGE}:\${env.BUILD_NUMBER} \
                    --destination \${BACKEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context \${WORKSPACE} \
                    --dockerfile \${WORKSPACE}/todo-frontend/Dockerfile \
                    --destination \${FRONTEND_IMAGE}:\${env.BUILD_NUMBER} \
                    --destination \${FRONTEND_IMAGE}:latest
                    """
                }
            }
        }
    }
}
