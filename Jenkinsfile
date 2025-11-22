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
                echo 'Building JAR..5....'
                sh './mvnw clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image....'
                sh "docker build -t petclinic:${BUILD_NUMBER} ."
            }
        }

        stage('Finished') {
            steps {
                echo 'CI pipeline completed successfully!!!!'
            }
        }
    }
}
