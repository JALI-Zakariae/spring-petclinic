pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Project cloned automatically.'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh './mvnw test'
                echo 'les tests passent'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh './mvnw sonar:sonar -Dsonar.projectKey=JALI-Zakariae_spring-petclinic -Dsonar.organization=jali-zakariae'
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Building JAR .........'
                sh './mvnw clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image....'
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
                            sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                            sh "docker tag petclinic:${BUILD_NUMBER} \$DOCKER_USER/petclinic:${BUILD_NUMBER}"
                            sh "docker push \$DOCKER_USER/petclinic:${BUILD_NUMBER}"
                        }
                    }
                }
       
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                    
                 
                    sh "kubectl set image deployment/petclinic petclinic=${DOCKER_IMAGE} -n ${KUBE_NAMESPACE}"
                    
                
                    sh "kubectl rollout status deployment/petclinic -n ${KUBE_NAMESPACE} --timeout=300s"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                echo 'Running smoke tests...'
                script {
                  
                    sh "sleep 30"
                    
                   
                    sh """
                        URL=\$(minikube service petclinic-service --url -n ${KUBE_NAMESPACE})
                        curl -f \$URL/actuator/health || exit 1
                    """
                }
            }
        }

        stage('Finished') {
            steps {
                echo 'CI pipeline completed successfully!!!!!!!'
            }
        }
    }
}
