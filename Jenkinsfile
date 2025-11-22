pipeline {
    agent any

    environment {
        // 1. Credentials
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        GITHUB_CREDS    = credentials('github-pat') // Make sure ID matches Step 1
        
        // 2. Configuration
        DOCKERHUB_USER  = "mohamadfawzi" // <--- CHANGE THIS
        APP_NAME        = "helloapp"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        GIT_REPO_URL    = "github.com/MOHAMADFAWZI-911/app-CI-CD.git" // <--- CHANGE THIS
    }

    stages {
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "🔨 Building Docker Image..."
                    sh "docker build -t $DOCKERHUB_USER/$APP_NAME:$IMAGE_TAG ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "📤 Pushing to DockerHub..."
                    // Login using the credentials variable
                    sh "echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin"
                    sh "docker push $DOCKERHUB_USER/$APP_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Update Manifest (GitOps)') {
            steps {
                script {
                    echo "📝 Updating Kubernetes Manifest..."
                    // Configure Git
                    sh """
                        git config user.email "jenkins@pipeline.com"
                        git config user.name "Jenkins Pipeline"
                    """
                    
                    // Update the deployment.yaml file
                    // This assumes your K8s file is at 'k8s/deployment.yaml'
                    // It looks for 'image: something' and replaces it with the new tag
                    sh """
                        sed -i 's|image: .*|image: $DOCKERHUB_USER/$APP_NAME:$IMAGE_TAG|' k8s/deployment.yaml
                    """
                    
                    // Commit and Push the change
                    withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh """
                            git add k8s/deployment.yaml
                            git commit -m "CI: Update image to version $IMAGE_TAG [skip ci]"
                            git push https://$GIT_USER:$GIT_PASS@$GIT_REPO_URL HEAD:main
                        """
                    }
                }
            }
        }
    }
}
