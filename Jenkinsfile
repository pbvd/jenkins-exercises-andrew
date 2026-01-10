@Library('shared-library') _  // Import our shared library!

pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'pbvld/nodejs-bootcamp-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }
    
    stages {
        stage('Increment Version') {
            steps {
                script {
                    dir('app') {
                        sh 'npm version minor --no-git-tag-version'
                        env.IMAGE_VERSION = sh(
                            script: 'node -p "require(\'./package.json\').version"',
                            returnStdout: true
                        ).trim()
                        echo "New version: ${env.IMAGE_VERSION}"
                    }
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    dir('app') {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dir('app') {
                        // Using shared library function! 🎉
                        buildDockerImage("${DOCKER_HUB_REPO}", "${env.IMAGE_VERSION}")
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        // Using shared library function! 🎉
                        pushDockerImage("${DOCKER_HUB_REPO}", "${env.IMAGE_VERSION}")
                    }
                }
            }
        }
        
        stage('Commit Version Change') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh '''
                            git config user.email "jenkins@pipeline.com"
                            git config user.name "Jenkins Pipeline"
                            git add app/package.json app/package-lock.json
                            git commit -m "chore: Bump version to ${IMAGE_VERSION}"
                            git push https://${GIT_TOKEN}@github.com/pbvd/jenkins-exercises-andrew.git HEAD:master
                        '''
                    }
                }
            }
        }
    }
    
    post {
        success {
            script {
                // Using shared library function! 🎉
                sendNotification('SUCCESS', "Pipeline completed! Version ${env.IMAGE_VERSION} deployed successfully.")
            }
        }
        failure {
            script {
                // Using shared library function! 🎉
                sendNotification('FAILURE', "Pipeline failed! Check logs for details.")
            }
        }
        unstable {
            script {
                sendNotification('UNSTABLE', "Build is unstable. Review test results.")
            }
        }
    }
}