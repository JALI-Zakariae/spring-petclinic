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
                bat 'mvn -U clean test'
                echo 'Running tests...'
                bat './mvnw test'
                echo 'les tests passent'
            }
        }

        stage('Package') {
            steps {
                echo 'Building JAR...'
                bat './mvnw clean package'
            }
        }


        
        stage('Finished') {
            steps {
                echo 'CI pipeline completed successfully!'
            }
        }
    }
}
