cd ~/k8s-gitops-demo
git checkout main

# Create updated Jenkinsfile with correct PATH
cat > Jenkinsfile <<'EOF'
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'prod'], description: 'Select environment')
    }
    
    environment {
        GIT_REPO = 'https://github.com/Gehna08/k8s-gitops-demo.git'
        KUBECONFIG = credentials('minikube-kubeconfig')
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Deploying to ${params.ENVIRONMENT} environment"
                    git branch: "${params.ENVIRONMENT}",
                        credentialsId: 'git-credentials',
                        url: "${GIT_REPO}"
                }
            }
        }
        
        stage('Validate Manifests') {
            steps {
                script {
                    sh """
                        echo "Validating Kubernetes manifests..."
                        find manifests/${params.ENVIRONMENT} -name '*.yaml' -exec echo "Checking: {}" \\;
                    """
                }
            }
        }
        
        stage('Deploy to Environment') {
            steps {
                script {
                    sh """
                        export KUBECONFIG=\${KUBECONFIG}
                        echo "kubectl location: \$(which kubectl)"
                        echo "Deploying to ${params.ENVIRONMENT} namespace..."
                        kubectl apply -f manifests/${params.ENVIRONMENT}/
                        kubectl rollout status deployment/polling-app -n ${params.ENVIRONMENT}
                        kubectl get pods -n ${params.ENVIRONMENT} -l app=polling-app
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sh """
                        export KUBECONFIG=\${KUBECONFIG}
                        echo "Verification for ${params.ENVIRONMENT}:"
                        kubectl get all -n ${params.ENVIRONMENT} -l app=polling-app
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "Deployment to ${params.ENVIRONMENT} completed successfully!"
        }
        failure {
            echo "Deployment to ${params.ENVIRONMENT} failed!"
        }
    }
}