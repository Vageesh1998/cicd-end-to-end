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
                    docker build -t abhishekf5/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    sh '''
                    echo 'Push to Repo'
                    docker push abhishekf5/cicd-e2e:${BUILD_NUMBER}
                    '''
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
                        # 1. Update the file inside the deploy/ directory path
                        cat deploy/deploy.yaml
                        sed -i "s/32/${BUILD_NUMBER}/g" deploy/deploy.yaml
                        cat deploy/deploy.yaml
                        
                        # 2. Stage the file with its correct path structure
                        git add deploy/deploy.yaml
                        git commit -m "Updated the deploy yaml | Jenkins Pipeline Build #${BUILD_NUMBER}"
                        
                        # 3. Clean and push using dynamic vault credentials back to your repository
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
