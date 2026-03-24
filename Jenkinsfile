pipeline {
    agent any

    environment {
        IMAGE_NAME = 'grunt0341/java-app'
    }

    stages {

        stage('Build JAR') {
            steps {
                dir('java-app') {
                    sh 'chmod +x mvnw'
                    sh './mvnw clean package'
                }
            }
        }

        stage('Analyze Code Quality') {
            steps {
                dir('java-app') {
                    withSonarQubeEnv('SonarQube') {
                        sh './mvnw sonar:sonar -Dsonar.projectKey=java-app -Dsonar.host.url=http://sonar:9000'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('java-app') {
                    sh 'docker build -t grunt0341/java-app:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push grunt0341/java-app:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'KUBECONFIG=/var/jenkins_home/.kube/config kubectl apply -f deployment.yaml --validate=false'
            }
        }
    }
}