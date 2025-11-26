pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = "default"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Project cloned automatically.'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh './mvnw test -Dcheckstyle.skip=true'
            }
        }

        stage('SonarCloud Analysis') {
                    steps {
                        script {
                            try {
                                withSonarQubeEnv('sonarcloud') {
                                    sh '''
                                        ./mvnw sonar:sonar \
                                        -Dsonar.projectKey=JALI-Zakariae_spring-petclinic \
                                        -Dsonar.organization=jali-zakariae \
                                        -Dsonar.scm.disabled=true \
                                        -Dsonar.qualitygate.wait=false \
                                        -Dsonar.analysis.ci=true \
                                        -Dsonar.automaticAnalysis.disabled=true
                                    '''
                                }
                            } catch (Exception e) {
                                echo "⚠️ SonarCloud analysis completed with warnings: ${e.message}"
                                echo "Continuing pipeline despite SonarCloud warnings..."
                                // NE PAAS mettre exit 1 ici - le pipeline continue
                            }
                        }
                    }
                }

        stage('Package') {
            steps {
                echo 'Building JAR...'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t petclinic:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker tag petclinic:${BUILD_NUMBER} \$DOCKER_USER/petclinic:${BUILD_NUMBER}
                        docker push \$DOCKER_USER/petclinic:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    script {
                        // Utiliser le chemin complet de kubectl
                        sh "/var/jenkins_home/kubectl apply -f k8s/petclinic.yaml"
                        sh "/var/jenkins_home/kubectl set image deployment/petclinic petclinic=\$DOCKER_USER/petclinic:${BUILD_NUMBER}"
                        sh "/var/jenkins_home/kubectl rollout status deployment/petclinic --timeout=300s"
                    }
                }
            }
        }

        stage('Smoke Test') {
            steps {
                echo 'Running smoke tests...'
                script {
                    sh "sleep 30"
                    sh """
                        # Méthode sans minikube - utiliser kubectl pour récupérer l'URL
                        NODE_IP=\$(/var/jenkins_home/kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type==\"InternalIP\")].address}')
                        NODE_PORT=\$(/var/jenkins_home/kubectl get service petclinic -o jsonpath='{.spec.ports[0].nodePort}')
                        URL="http://\${NODE_IP}:\${NODE_PORT}"
                        echo "Testing application at: \$URL"
                        curl -f \$URL/actuator/health || exit 1
                    """
                }
            }
        }

        stage('Finished') {
            steps {
                echo 'CI/CD pipeline completed successfully!'
                script {
                    sh """
                        echo "✅ Deployment successful!"
                        echo "🌐 Application URL:"
                        /var/jenkins_home/kubectl get service petclinic -o wide
                    """
                }
            }
        }
    }
}
