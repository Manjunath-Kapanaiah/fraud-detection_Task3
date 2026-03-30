pipeline {
    agent { label 'JENKINS-2' }

    environment {
        GITLAB_TOKEN = credentials('gitlab-token')
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {

        stage('Check GitLab Pipeline Status') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitUntil {
                            def response = sh(
                                script: """
                                curl --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" https://gitlab.com/api/v4/projects/80702349/pipelines?per_page=1
                                """,
                                returnStdout: true
                            ).trim()

                            return response.contains('"status":"success"')
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f fraud-deployment.yaml
                    '''
                }
            }
        }

        stage('Trigger GitHub Actions') {
            steps {
                sh """
                curl -X POST -H "Authorization: Bearer ${GITHUB_TOKEN}" -H "Accept: application/vnd.github+json" https://api.github.com/repos/Manjunath-Kapanaiah/fraud-detection_Task3/actions/workflows/deploy.yml/dispatches -d '{"ref":"main"}'
                """
            }
        }

        stage('Wait for GitHub Workflow') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitUntil {
                            def result = sh(
                                script: """
                                curl -H "Authorization: Bearer ${GITHUB_TOKEN}" https://api.github.com/repos/Manjunath-Kapanaiah/fraud-detection_Task3/actions/runs | jq '.workflow_runs[0].conclusion'
                                """,
                                returnStdout: true
                            ).trim()

                            return result.contains("success")
                        }
                    }
                }
            }
        }
    }

    post {
        failure {
            steps {
                echo "Failure detected. Rolling back..."

                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl rollout undo deployment/fraud-detection
                    '''
                }

                sh """
                curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" https://api.github.com/repos/Manjunath-Kapanaiah/fraud-detection_Task3/issues -d '{"title":"Deployment Failed","body":"Rollback executed","labels":["incident"]}'
                """
            }
        }

        success {
            steps {
                echo "Pipeline completed successfully"
            }
        }
    }
}
