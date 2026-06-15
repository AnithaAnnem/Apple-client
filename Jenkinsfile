pipeline {
    agent any

    environment {
        NOMAD_ADDR = "http://127.0.0.1:4646"
        TEMPLATE_DIR = "/opt/nomad-templates"
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '0.1', description: 'Target Docker Image Tag to deploy')
    }

    stages {
        stage('Initialize Deployment Configuration') {
            steps {
                echo "Deploying target image tag: ${params.IMAGE_TAG}"
                echo "Tracking automation via Jenkins Build Run: ${env.BUILD_NUMBER}"
            }
        }

        stage('Inject Dynamic Corporate Configs') {
            steps {
                echo "Replacing production placeholders with runtime token keys..."
                
                // We wrap this stage inside credentials to safely pull the Artifactory Token alongside the Nomad variables
                withCredentials([
                    string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN'),
                    string(credentialsId: 'artifactory-token', variable: 'ART_TOKEN')
                ]) {
                    // Multi-expression sed command to swap DEPLOY_VERSION, RIO_BUILD_NUMBER, and ARTIFACTORY_TOKEN_VALUE
                    sh """
                        sed -e 's/DEPLOY_VERSION/${params.IMAGE_TAG}/g' \
                            -e 's/RIO_BUILD_NUMBER/${env.BUILD_NUMBER}/g' \
                            -e 's/ARTIFACTORY_TOKEN_VALUE/${ART_TOKEN}/g' \
                            ${env.TEMPLATE_DIR}/webapp.nomad.tpl > webapp.nomad
                    """
                }
            }
        }

        stage('Secure Deploy to Nomad') {
            steps {
                withCredentials([string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN')]) {
                    echo "Authenticating via ACL token and submitting deployment payload..."
                    // This now deploys using the newly generated file with all production settings
                    sh "nomad job run webapp.nomad"
                }
            }
        }

        stage('Verify Rollout Status') {
            steps {
                withCredentials([string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN')]) {
                    echo "Fetching secure corporate job allocation statuses..."
                    // Note: Changed target string from 'demo-webapp' to match his job identifier name 'v1-job-name'
                    sh "nomad job status v1-job-name"
                }
            }
        }
    }
}
