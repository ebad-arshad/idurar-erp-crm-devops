@Library('shared') _

pipeline {
    agent { label 'slave' }

    stages {
        stage('Version Calculation') {
            steps {
                script {
                    // Get current build number
                    def buildNumber = env.BUILD_NUMBER.toInteger()

                    // Calculate major and minor versions
                    def majorVersion = (buildNumber / 50).intValue()
                    def minorVersion = buildNumber % 50

                    // Format minor version with leading zero if needed
                    def formattedMinor = String.format('%02d', minorVersion)

                    // Create the tag
                    def imageTag = "${majorVersion}.${formattedMinor}"

                    // Store in environment variable
                    env.IMAGE_TAG = imageTag.toString()
                }
            }
        }
        stage('Clone') {
            steps {
                cleanWs()
                
                script {

                    // clone('https://github.com/ebad-arshad/idurar-erp-crm-devops', 'master')
                    // clone('https://github.com/ebad-arshad/idurar-erp-crm-devops', 'k8s')

                    sh "git clone --depth 1 -b master https://github.com/ebad-arshad/idurar-erp-crm-devops master"

                    sh "git clone --depth 1 -b k8s https://github.com/ebad-arshad/idurar-erp-crm-devops k8s"
                }
            }
        }
        stage('Login DockerHub') {
            steps {
                script {
                    dockerhub_login('docker-hub-creds')
                }
            }
        }
        stage('Build') {
            steps {
                echo 'This is build stage'
                sh "docker build -t ebadarshad/erp-frontend:${env.IMAGE_TAG} master/frontend"
                sh "docker build -t ebadarshad/erp-backend:${env.IMAGE_TAG} master/backend"
                echo 'Build images successful'
            }
        }
        // stage('Security Scan with Trivy') {
        //     steps {
        //         sh "trivy image --severity HIGH,CRITICAL --exit-code 1 --format json -o trivy-report.json ebadarshad/erp-frontend:${env.IMAGE_TAG}"
        //         sh "trivy image --severity HIGH,CRITICAL --exit-code 1 --format json -o trivy-report.json ebadarshad/erp-backend:${env.IMAGE_TAG}"
        //         archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
        //     }
        // }
        stage('Push image to DockerHub') {
            steps {
                echo 'This is Dockerhub image push stage'
                sh "docker push ebadarshad/erp-frontend:${env.IMAGE_TAG}"
                sh "docker push ebadarshad/erp-backend:${env.IMAGE_TAG}"
                echo 'Pushed image to Dockerhub'
            }
        }
        stage('Update K8s Manifests in GitHub') {
            steps {
                script {
                    dir('k8s') {
                        // Commit and push to GitHub
                        sh """
                            git config user.email 'm.ebadarshad2003@gmail.com'
                            git config user.name 'ebad-arshad'
                            sed -i 's|ebadarshad/erp-frontend:[^ ]*|ebadarshad/erp-frontend:${env.IMAGE_TAG}|g' deployment.yaml
                            sed -i 's|ebadarshad/erp-backend:[^ ]*|ebadarshad/erp-backend:${env.IMAGE_TAG}|g' deployment.yaml
                    
                            git add deployment.yaml
                    
                        """
                        withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                            sh """
                                if ! git diff --cached --quiet; then
                                    git commit -m 'ci: update image tags to ${env.IMAGE_TAG}'
                                    
                                    git push https://${GIT_TOKEN}@github.com/${GIT_USER}/idurar-erp-crm-devops.git HEAD:k8s
                                fi
                            """
                        }
                    }
                }
            }
        }
    }
}
