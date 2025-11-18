pipeline {
    agent {
            docker {
                image 'maven:3.9.0-openjdk-17'
                args '-v /var/run/docker.sock:/var/run/docker.sock'
            }
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
                echo 'Building JAR...'
                sh './mvnw clean package'
            }
        }
        stage('Docker Build') {
                    steps {
                        echo 'Building Docker image...'
                        sh "docker build -t petclinic:${BUILD_NUMBER} ."
                        sh "docker run -d -p 9090:9090 --name petclinic-test petclinic:${BUILD_NUMBER}"
                    }
                }

        
        stage('Finished') {
            steps {
                echo 'CI pipeline completed successfully!!!!'
            }
        }
    }
}
