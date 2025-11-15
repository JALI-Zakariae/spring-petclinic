pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                echo 'Cloning project...'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Finished') {
            steps {
                echo 'Pipeline completed successfully!'
            }
        }
    }
}
