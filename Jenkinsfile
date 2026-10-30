
pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/aishwaryan153/proj-3-aws-jenkins-k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t proj3:latest .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag proj3:latest aishwaryalaxmi0599/proj3:latest'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push aishwaryalaxmi0599/proj3:latest'
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'k8s-token',
                        variable: 'K8S_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        set -e

                        cat > kubeconfig-test <<EOF
apiVersion: v1
kind: Config
clusters:
- name: eks-cluster
  cluster:
    server: ${endpoint}
    certificate-authority-data: ${cert}
users:
- name: jenkins-sa
  user:
    token: ${K8S_TOKEN}
contexts:
- name: jenkins-context
  context:
    cluster: eks-cluster
    user: jenkins-sa
    namespace: web-apps
current-context: jenkins-context
EOF

                        chmod 600 kubeconfig-test
                        export KUBECONFIG="$PWD/kubeconfig-test"

                        echo "Deploying new Docker image..."

                        kubectl set image deployment/proj3-deployment \
                        proj3-container=aishwaryalaxmi0599/proj3:latest \
                        -n web-apps

                        echo "Waiting for rollout..."

                        kubectl rollout status deployment/proj3-deployment \
                        -n web-apps --timeout=180s

                        echo "Deployment successful!"

                        rm -f kubeconfig-test
                    '''
                }
            }

            post {
                always {
                    sh 'rm -f kubeconfig-test'
                }
            }
        }
    }
}
