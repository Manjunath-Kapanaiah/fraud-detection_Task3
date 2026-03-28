pipeline {
    agent { label 'K8s' }

    stages {

        stage('Fraud Detection') {
            steps {
                echo "Fraud detection completed"
            }
        }

        stage('Trigger GitLab') {
            steps {
                sh '''
                curl --request POST \
                  --form token=Token \
                  --form ref=main \
                  https://gitlab.com/Manjunath-Kapanaiah/user-auth-service.git
                '''
            }
        }
    }
}
