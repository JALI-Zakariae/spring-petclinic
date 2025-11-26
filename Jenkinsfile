pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Project cloned automatically'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh './mvnw test'
            }
        }

        stage('Package') {
            steps {
                echo 'Compiling and packaging the app...'
                sh './mvnw clean package'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        ${tool('SonarScanner')}/bin/sonar-scanner \
                        -Dsonar.projectKey=YassineZitouni29_spring-petclinic \
                        -Dsonar.organization=yassinezitouni29 \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.host.url=https://sonarcloud.io
                    """
                }
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
                        credentialsId: 'dockerhub',
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
        stage('Kubernetes Deploy') {
            steps {
                echo "Deploying latest image to Kubernetes..."

                // Replace the container image inside the existing Deployment
                sh """
                    kubectl set image deployment/petclinic \
                        petclinic=$DOCKER_USER/petclinic:${BUILD_NUMBER} \
                        --record
                """
            }
        }


        stage('Finished') {
            steps {
                echo 'CI pipeline completed successfully!'
            }
        }
    }
}
