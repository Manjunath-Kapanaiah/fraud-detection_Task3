pipeline {
    agent any
    environment {
        // K8s commands are mocked using echo or Docker simulation
        KUBECTL_CMD = 'echo "kubectl mock command"'
        DEPLOYMENT_NAME = 'fraud-app'
    }

    stages {

        stage('Checkout Fraud Detection') {
            steps {
                git branch: 'main', url: 'git@github.com:Manjunath-Kapanaiah/Fraud-Detection.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Use Python virtual environment
                    python3 -m venv venv || echo "venv not created"
                    source venv/bin/activate || echo "activate venv failed"
                    pip install -r requirements.txt || echo "pip install skipped"
                '''
            }
        }

        stage('Unit & Integration Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'echo "Running unit tests..."'
                        // Replace actual tests with mock
                        sh 'sleep 2'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'echo "Running integration tests..."'
                        // Replace actual integration with mock
                        sh 'sleep 2'
                    }
                }
            }
        }

        stage('Deploy (Mock)') {
            steps {
                script {
                    // Instead of real kubectl, we simulate deployment
                    sh "${env.KUBECTL_CMD} apply -f deployment.yaml || echo 'Mock deploy skipped'"
                }
            }
        }

        stage('Trigger Next Pipelines') {
            steps {
                script {
                    echo "Triggering GitLab User Auth Pipeline..."
                    sh """
                        curl -X POST -H "PRIVATE-TOKEN: token" \
                        "https://gitlab.com/Manjunath-Kapanaiah/user-auth-service.git" \
                        || echo 'GitLab trigger mocked'
                    """

                    echo "Triggering GitHub Transaction Service..."
                    sh """
                        curl -X POST https://api.github.com/repos/Manjunath-Kapanaiah/transaction-service/dispatches \
                        -H "Accept: application/vnd.github+json" \
                        -H "Authorization: Bearer token" \
                        -d '{"event_type":"fraud-check","client_payload":{"status":"success"}}' \
                        || echo 'GitHub dispatch mocked'
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed, performing mock rollback..."
            sh "${env.KUBECTL_CMD} rollout undo deployment/${DEPLOYMENT_NAME} || echo 'Rollback mocked'"
            
            echo "Notifying teams via GitHub Issues..."
            sh """
                curl -X POST -H "Authorization: token Token" \
                https://github.com/Manjunath-Kapanaiah/fraud-detection_Task3.git \
                -d '{"title":"Rollback Alert","body":"Pipeline failed. Fraud Detection rolled back (mock)."}' \
                || echo 'Issue creation mocked'
            """
        }
        success {
            echo "Pipeline succeeded. No rollback needed."
        }
    }
}
