pipeline {
    agent any

    tools {
        // EXACTEMENT le nom défini dans Global Tool Configuration
        maven 'Maven_3_9'
    }

    stages {
        stage('Build & Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
    }

    post {
        success {
            echo '✅ Build réussi (tests ignorés).'
        }
        failure {
            echo '❌ Build cassé, va voir les logs.'
        }
    }
}
