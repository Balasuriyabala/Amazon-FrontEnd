# Amazon-FE

For Deployment rollback
pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to rollback from')
        string(name: 'TAG', defaultValue: 'v1.0', description: 'Git tag to rollback to')
    }

    stages {
        stage('Checkout Branch') {
            steps {
                git url: 'https://your-repo.git', branch: "${params.BRANCH}"
            }
        }

        stage('Checkout Tag') {
            steps {
                sh """
                    git fetch --tags
                    git checkout tags/${params.TAG}
                """
            }
        }

        stage('Validate Tag Belongs to Branch') {
            steps {
                script {
                    def result = sh(script: "git merge-base --is-ancestor ${params.TAG} ${params.BRANCH}", returnStatus: true)
                    if (result != 0) {
                        error "Tag ${params.TAG} does not belong to branch ${params.BRANCH}. Rollback aborted."
                    }
                }
            }
        }

        stage('Confirm Version') {
            steps {
                sh "git log -1 --oneline"
            }
        }
    }
}
