pipeline {
    agent any

    environment {
        // Set API keys and other environment variables here
        VITE_APP_API_ENDPOINT_URL = 'https://api.themoviedb.org/3'
        VITE_APP_TMDB_V3_API_KEY = '4814d2daa04521fab8b5534221ff0038'
        TMDB_API_KEY = "4814d2daa04521fab8b5534221ff0038"
        IMAGE_NAME = "prevasishta98/netflix-clone"
        IMAGE_TAG = "latest"
    }

    stages {
        // Stage 1: Clone Repository
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/prerana-vasishta/Netflix-clone.git'
            }
        }

        // Stage 2: Build and Deploy React App (Bare Metal)
        stage('Build Vite App on Bare Metal') {
            steps {
                // Create the .env file with the necessary environment variables
                sh '''
                    echo "VITE_APP_API_ENDPOINT_URL=$VITE_APP_API_ENDPOINT_URL" > .env
                    echo "VITE_APP_TMDB_V3_API_KEY=$VITE_APP_TMDB_V3_API_KEY" >> .env
                    cat .env
                '''
                // Install dependencies and build the app
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Serve App on Bare Metal') {
            steps {
                // Serve the app on port 3000
                sh 'npx serve -s dist -l 3000 &'
            }
        }

        // Stage 3: Docker Build & Deploy
        stage('Docker Build') {
            steps {
                // Build the Docker image with the API key as a build argument
                sh '''
                    docker build \
                    --build-arg TMDB_V3_API_KEY=${TMDB_API_KEY} \
                    -t netflix-clone .
                '''
            }
        }

        stage('Docker Run') {
            steps {
                // Stop any existing container and run the new Docker container
                sh '''
                    docker ps -q --filter "ancestor=netflix-clone" | xargs -r docker stop
                    docker run -d -p 8081:80 netflix-clone
                '''
            }
        }

        // Stage 4: Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl set image deployment/netflix-app netflix-app=${IMAGE_NAME}:${IMAGE_TAG}
                        kubectl apply -f Kubernetes/service.yml
                    '''
                }
            }
        }

        // Stage 5: Access Info (for Kubernetes)
        stage('Access Info') {
            steps {
                withCredentials([file(credentialsId: 'minikube-config', variable: 'KUBECONFIG')]) {
                    sh '''
                        echo "Forwarding port 9090 to service port 80"
                        kubectl port-forward svc/netflix-app 9090:30007 &
                    '''
                }
                echo "✅ Deployment Successful!"
                echo "🌐 Access your Netflix Clone at: http://localhost:9090"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
