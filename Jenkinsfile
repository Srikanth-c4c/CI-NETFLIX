stage('Update Kubernetes Manifest') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'srikanth-git', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                sh '''
                    set -ex
                    rm -rf CD-Netflix manifest

                    # Clone the GitHub repo containing Kubernetes manifests
                    git clone -b ${MANIFEST_BRANCH} https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Srikanth-c4c/CD-Netflix.git

                    # ✅ Correct navigation
                    cd CD-Netflix/manifest
                    ls -la

                    # Validate the manifest file exists
                    if [ ! -f ${DEPLOY_FILE} ]; then
                        echo "Error: ${DEPLOY_FILE} not found!"
                        exit 1
                    fi

                    # Display the current image line
                    echo "Current image line:"
                    grep 'image:' ${DEPLOY_FILE}

                    # ✅ Update the image tag with the new one
                    sed -i "s#image: .*#image: ${DOCKER_REPO}/netflix:${IMAGE_TAG}#" ${DEPLOY_FILE}

                    # Verify the change
                    echo "Updated image line:"
                    grep 'image:' ${DEPLOY_FILE}

                    # Commit and push changes
                    git config user.name "Srikanth-c4c"
                    git config user.email "srikanth@gmail.com"    # Replace with your GitHub email
                    git add ${DEPLOY_FILE}
                    git commit -m "Update image tag to ${IMAGE_TAG}"
                    git push origin ${MANIFEST_BRANCH}
                '''
            }
        }
    }
}
