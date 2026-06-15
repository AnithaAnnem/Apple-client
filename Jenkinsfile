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
        stage('Initialize & Validate Template') {
            steps {
                echo "Deploying target image tag: ${params.IMAGE_TAG}"
                echo "Tracking automation via Jenkins Build Run: ${env.BUILD_NUMBER}"
                
                // Pre-flight check: Ensure the template file exists before running sed
                sh "test -f ${env.TEMPLATE_DIR}/webapp.nomad.tpl || (echo 'ERROR: Source template missing!'; exit 1)"
            }
        }

        stage('Inject Dynamic Corporate Configs') {
            steps {
                echo "Replacing production placeholders with runtime token keys..."
                
                withCredentials([string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN')]) {
                    // Replaces placeholders and leaves static blocks (update, migrate, restart) intact
                    sh """
                        sed -e 's/DEPLOY_VERSION/${params.IMAGE_TAG}/g' \
                            -e 's/RIO_BUILD_NUMBER/${env.BUILD_NUMBER}/g' \
                            -e 's/ARTIFACTORY_TOKEN_VALUE/NOT_REQUIRED/g' \
                            ${env.TEMPLATE_DIR}/webapp.nomad.tpl > webapp.nomad
                    """
                }
            }
        }

        stage('Secure Deploy to Nomad') {
            steps {
                withCredentials([string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN')]) {
                    echo "Submitting deployment payload..."
                    
                    // Validate job syntax locally before pushing to the cluster api
                    sh "nomad job validate webapp.nomad"
                    
                    // Run the job execution plan
                    sh "nomad job run webapp.nomad"
                }
            }
        }

        stage('Verify Rollout & Migration Status') {
            steps {
                withCredentials([string(credentialsId: 'nomad-acl-token', variable: 'NOMAD_TOKEN')]) {
                    echo "Monitoring update progress against healthy_deadline policies..."
                    
                    // Tracks the rolling deployment live. If the update stalls or violates 
                    // progress_deadline (25m) or healthy_deadline (20m), Jenkins will catch the failure.
                    sh "nomad job status -monitor v1-job-name"
                }
            }
        }
    }
}
