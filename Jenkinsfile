pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    command: [ "/busybox/cat" ]
    tty: true
    volumeMounts:
      - name: kaniko-conf
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-conf
      emptyDir: {}
"""
        }
    }

    environment {
        HARBOR_REGISTRY = "harbor.bilgem.tubitak.gov.tr"
        HARBOR_PROJECT  = "tmp"
        // Hesap bilgilerim
        HARBOR_USER     = "idilsu.elmas" 
        HARBOR_PASS     = "GokayEldemdeas173422.*"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/delavenka/jenkins.git'
            }
        }

        stage('Create Harbor Config') {
            steps {
                container('kaniko') {
                    // Kullanıcı adı ve şifreyi birleştirip Base64'e çeviriyoruz
                    sh """
                    AUTH_BASE64=\$(echo -n "\${HARBOR_USER}:\${HARBOR_PASS}" | base64 | tr -d '\\n')
                    echo "{\\\"auths\\\":{\\\"\${HARBOR_REGISTRY}\\\":{\\\"auth\\\":\\\"\$AUTH_BASE64\\\"}}}" > /kaniko/.docker/config.json
                    """
                    echo "Kaniko auth dosyasi oluşturuldu."
                }
            }
        }

        stage('Build & Push Backend') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context=${WORKSPACE}/todo-backend \
                    --dockerfile=${WORKSPACE}/todo-backend/Dockerfile \
                    --destination=${HARBOR_REGISTRY}/${HARBOR_PROJECT}/todo-backend:latest \
                    --insecure \
                    --skip-tls-verify \
                    --snapshot-mode=redo \
                    --use-new-run \
                    --compression=zstd \
                    --compression-level=1
                    """
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context=${WORKSPACE}/todo-frontend \
                    --dockerfile=${WORKSPACE}/todo-frontend/Dockerfile \
                    --destination=${HARBOR_REGISTRY}/${HARBOR_PROJECT}/todo-frontend:latest \
                    --insecure \
                    --skip-tls-verify \
                    --snapshot-mode=redo \
                    --use-new-run \
                    --compression=zstd \
                    --compression-level=1
                    """
                }
            }
        }
    }
}
