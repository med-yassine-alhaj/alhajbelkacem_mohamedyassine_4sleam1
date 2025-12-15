pipeline {
    agent any

    environment {
        M2_HOME = "/usr/share/maven"
        PATH = "${env.M2_HOME}/bin:${env.PATH}"

        IMAGE_NAME = "medyassineelhaj/student-management"
        DOCKERHUB_CREDENTIALS = "dockerhub-cred"
        K8S_CREDENTIALS = "k8s-cred"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/med-yassine-alhaj/alhajbelkacem_mohamedyassine_4sleam1.git'
            }
        }

        stage('Test') {
            steps {
               sh 'mvn clean verify'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                      mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                        -Dsonar.projectName=student-management
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: DOCKERHUB_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: K8S_CREDENTIALS, variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        kubectl version --client
                        kubectl get nodes
                        kubectl set image deployment/spring-app spring-app=${IMAGE_NAME}:latest -n devops
                        kubectl rollout status deployment spring-app -n devops
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
