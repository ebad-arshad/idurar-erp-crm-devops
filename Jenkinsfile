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
                script {
                    sh "rm -rf *"

                    clone('https://github.com/ebad-arshad/idurar-erp-crm-devops', 'master')
                    // git clone -b master https://github.com/ebad-arshad/idurar-erp-crm-devops master

                    clone('https://github.com/ebad-arshad/idurar-erp-crm-devops', 'k8s')
                    // git clone -b k8s https://github.com/ebad-arshad/idurar-erp-crm-devops k8s
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
                    sh "cd k8s"
                    // Commit and push to GitHub
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            git config user.email 'm.ebadarshad2003@gmail.com'
                            git config user.name 'ebad-arshad'
                            ls
                            sed -i 's|ebadarshad/erp-frontend:[^ ]*|ebadarshad/erp-frontend:${env.IMAGE_TAG}|g' deployment.yaml
                            sed -i 's|ebadarshad/erp-backend:[^ ]*|ebadarshad/erp-backend:${env.IMAGE_TAG}|g' deployment.yaml
                            git add deployment.yaml
                            if git diff --cached --quiet; then
                                echo 'No changes to commit'
                            else
                                git commit -m 'ci: update image tags to ${env.IMAGE_TAG}'
                                git push https://${GIT_TOKEN}@github.com/ebad-arshad/idurar-erp-crm-devops.git
                            fi
                        """
                    }
                }
            }
        }
        // stage('Deploy') {
        //     steps {
        //         echo 'This is deploy stage'
        //         sh 'docker compose down'
        //         sh "APP_VERSION=${env.IMAGE_TAG} docker compose up --build -d"
        //         echo "Deployed version: ${env.IMAGE_TAG}"

        //         // sh """
        //         // cd ./k8s 
        //         // kubectl apply -f namespace.yaml
        //         // kubectl apply -f secret.yaml
        //         // kubectl apply -f pv.yaml
        //         // kubectl apply -f pvc.yaml
        //         // kubectl apply -f service.yaml
        //         // kubectl apply -f hpa.yaml
        //         // kubectl apply -f statefulset.yaml
        //         // kubectl apply -f deployment.yaml 

        //         // Thread.sleep(10000)

        //         // kubectl port-forward svc/erp-frontend -n smp 80:80 --address=0.0.0.0 &               
        //         // """
        //     }
        // }
    }
}
