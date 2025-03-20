pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'srikanthk419/netflix'                           // Your Docker image name
        DOCKER_REPO = 'srikanthk419'                           // Your Docker Hub repo
        MANIFEST_REPO = 'https://github.com/Srikanth-c4c/CD-Netflix.git'  // Updated manifest repo
        MANIFEST_BRANCH = 'main'                               // Branch name
        DEPLOY_FILE = 'deployment.yml'                   // Path to the manifest file
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'ls'
                }
            }
        }

        stage('Get Git Commit ID') {
            steps {
                script {
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Using Git commit ID as tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'srikanth-docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo '${DOCKER_PASSWORD}' | docker login -u '${DOCKER_USERNAME}' --password-stdin
                        """
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    sh """
                        set -ex
                        if [ ! -d client ]; then echo "Error: client directory not found"; exit 1; fi
                        cd CI-NETFLIX
                        ls
                        docker build --no-cache -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_REPO}/${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}/${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'srikanth-git', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                            set -ex
                            
                            # Remove existing repo if exists
                            rm -rf .

                            # Clone the Git repo containing Kubernetes manifests using credentials
                            git clone -b ${MANIFEST_BRANCH} https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Srikanth-c4c/CD-Netflix.git
                            cd CD-Netflix

                            # Update image tag in deploy.yml
                            sed -i 's|image: ${DOCKER_REPO}/${DOCKER_IMAGE}:.*|image: ${DOCKER_REPO}/${DOCKER_IMAGE}:${IMAGE_TAG}|' ${DEPLOY_FILE}

                            # Commit and push changes
                            git config user.name "Srikanth-c4c"
                            git config user.email "your-email@example.com"
                            git add ${DEPLOY_FILE}
                            git commit -m "Update image tag to ${IMAGE_TAG}"
                            git push origin ${MANIFEST_BRANCH}
                        """
                    }
                }
            }
        }
    }
}
