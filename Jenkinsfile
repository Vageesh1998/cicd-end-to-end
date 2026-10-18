pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'github-token', 
                url: 'https://github.com/Vageesh1998/cicd-end-to-end',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t vageeshhiremath/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push Artifacts to DockerHub') {
            steps {
                script {
                    // Uses your newly created Docker Hub write-token credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // 1. Securely log in to Docker Hub using standard input streams
                        sh "echo \${DOCKER_PASSWORD} | docker login -u \${DOCKER_USERNAME} --password-stdin"
                        
                        // 2. Push the built image tagged with the unique build number
                        sh "docker push \${DOCKER_USERNAME}/cicd-e2e:\${BUILD_NUMBER}"
                        
                        // 3. Log out immediately to clean up secure session tokens from the workspace
                        sh "docker logout"
                    }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github-token', 
                url: 'https://github.com/Vageesh1998/cicd-end-to-end.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
            
                        cat deploy/deploy.yaml
                        sed -i "s/32/${BUILD_NUMBER}/g" deploy/deploy.yaml
                        cat deploy/deploy.yaml
                        
                        git add deploy/deploy.yaml
                        git commit -m "Updated the deploy yaml | Jenkins Pipeline Build #${BUILD_NUMBER}"
                        
                        git remote rm origin-fork || true
                        git remote add origin-fork https://${GIT_USERNAME}:${GIT_PASSWORD}@://github.com
                        git push origin-fork HEAD:main
                        '''                    
                    }
                }
            }
        }
    }
}
