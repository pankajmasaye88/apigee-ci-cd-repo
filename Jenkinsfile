pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_TYPE', choices: ['apiproxy', 'sharedflow'], description: 'Deploy a Proxy or Shared Flow')
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'stage', 'prod'], description: 'Target Apigee Environment')
        string(name: 'API_NAME', defaultValue: 'sample-proxy', description: 'Name of the Proxy or Shared Flow')
    }

    environment {
        PROJECT_ID = 'your-gcp-project-id'
        SERVICE_ACCOUNT = credentials('GCP_SA_JSON')
    }

    tools {
        nodejs 'NodeJS_18'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-org/your-apigee-repo.git'
            }
        }

        stage('Auth GCP') {
            steps {
                withCredentials([file(credentialsId: 'GCP_SA_JSON', variable: 'GCLOUD_KEY')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GCLOUD_KEY'
                    sh 'gcloud config set project $PROJECT_ID'
                }
            }
        }

        stage('Install Tools') {
            steps {
                sh 'curl -sL https://github.com/apigee/apigeecli/releases/latest/download/apigeecli-linux -o apigeecli'
                sh 'chmod +x apigeecli && sudo mv apigeecli /usr/local/bin/'
                sh 'npm install -g apigeelint'
            }
        }

        stage('Lint Code') {
            steps {
                script {
                    def path = params.DEPLOY_TYPE == 'apiproxy' ? 'apiproxy' : 'sharedflowbundle'
                    sh "apigeelint -s ${path} -f stylish -c tests/apigeelint_rules.json || true"
                }
            }
        }

        stage('Import & Deploy to Apigee') {
            steps {
                script {
                    def token = sh(script: 'gcloud auth print-access-token', returnStdout: true).trim()
                    def path = params.DEPLOY_TYPE == 'apiproxy' ? 'apiproxy' : 'sharedflowbundle'
                    def importCmd = params.DEPLOY_TYPE == 'apiproxy'
                        ? "apigeecli apis import --name ${params.API_NAME} --file ${path} --token ${token}"
                        : "apigeecli sharedflows import --name ${params.API_NAME} --file ${path} --token ${token}"
                    def revisionCmd = params.DEPLOY_TYPE == 'apiproxy'
                        ? "apigeecli apis get --name ${params.API_NAME} --token ${token} | jq -r '.revision[-1]'"
                        : "apigeecli sharedflows get --name ${params.API_NAME} --token ${token} | jq -r '.revision[-1]'"
                    def deployCmd = params.DEPLOY_TYPE == 'apiproxy'
                        ? "apigeecli apis deploy --name ${params.API_NAME} --env ${params.TARGET_ENV} --rev REV --token ${token}"
                        : "apigeecli sharedflows deploy --name ${params.API_NAME} --env ${params.TARGET_ENV} --rev REV --token ${token}"

                    sh importCmd
                    def rev = sh(script: revisionCmd, returnStdout: true).trim()
                    sh deployCmd.replace('REV', rev)
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed!"
        }
        failure {
            echo "❌ Deployment failed. Check console logs."
        }
    }
}
