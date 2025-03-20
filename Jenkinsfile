pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'srikanthk419/netflix'                  // Docker image name
        DOCKER_REPO = 'srikanthk419'                           // Docker Hub repo
        MANIFEST_REPO = 'https://github.com/Srikanth-c4c/CD-Netflix.git'  // Manifest repo
        MANIFEST_BRANCH = 'main'                               // Branch name
        DEPLOY_FILE = 'deployment.yml'                         // Manifest file path
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

                        # Ensure CI-NETFLIX directory exists
                       # if [ ! -d CI-NETFLIX ]; then echo "Error: CI-NETFLIX directory not found"; exit 1; fi

                        # Navigate to CI-NETFLIX directory
                        pwd
			ls
                        cd /var/lib/jenkins/workspace/netflix-git-ops/
                        ls

                        # Docker build
                        docker build --no-cache -t ${DOCKER_REPO}/netflix:${IMAGE_TAG} .

                        # Tag and push the image
                        docker push ${DOCKER_REPO}/netflix:${IMAGE_TAG}
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

                            # Remove old manifests repo if exists
                            rm -rf CD-Netflix

                            # Clone the GitHub repo containing Kubernetes manifests
                            git clone -b ${MANIFEST_BRANCH} https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Srikanth-c4c/CD-Netflix.git
                            cd CD-Netflix

                            # Update image tag in deployment.yml
                            sed -i 's|image: ${DOCKER_REPO}/netflix:.*|image: ${DOCKER_REPO}/netflix:${IMAGE_TAG}|' ${DEPLOY_FILE}

                            # Commit and push changes
                            git config user.name "Srikanth-c4c"
                            git config user.email "your-email@example.com"  # Replace with your GitHub email
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

