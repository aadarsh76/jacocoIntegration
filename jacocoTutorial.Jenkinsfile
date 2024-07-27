pipeline {
    agent any

    environment {
        JACOCO_AGENT_VERSION = '0.8.3' // Specify the JaCoCo version
    }

    stages {
        stage('Prepare Docker Configuration') {
            steps {
                script {
                    // Define paths
                    def dockerComposeFile = 'docker-compose.yml'
                    def dockerComposeBackupFile = 'docker-compose_bkp.yml'
                    def jacocoAgentPath = '/root/jacoco_agent/org.jacoco.agent-' + JACOCO_AGENT_VERSION + '-runtime.jar'
                    def jacocoExecFile = '/root/jacoco_agent/jacoco-application.exec'
                    
                    // Backup existing Docker Compose file
                    sh "cp ${dockerComposeFile} ${dockerComposeBackupFile}"

                    // Update Docker Compose file to include JaCoCo agent
                    sh """
                        sed -i -e 's|\\("-jar", "/root/application.jar"\\)|"-javaagent:${jacocoAgentPath}=destfile=${jacocoExecFile}", \\1|g' ${dockerComposeFile}
                    """

                    // Add volume for JaCoCo agent data
                    sh """
                        awk 'BEGIN {services=0; app=0; volumes=0; done=0}
                        /^services:/ { services=1; print; next }
                        services && /^  app:$/ { app=1; print; next }
                        services && !/^  / { app=0; volumes=0; }
                        app && /^    volumes:$/ { volumes=1; print; print "      - \\"/var/jacoco_agent:/root/jacoco_agent\\""; done=1; next }
                        app && volumes && done { print; next }
                        { print }' ${dockerComposeFile} > ${dockerComposeBackupFile}_new.yml
                        mv ${dockerComposeBackupFile}_new.yml ${dockerComposeFile}
                    """
                }
            }
        }

        stage('Restart Docker Services') {
            steps {
                script {
                    // Restart Docker services to apply changes
                    sh """
                        docker-compose down || true
                        sleep 5m
                        docker-compose up -d
                        sleep 5m
                    """
                }
            }
        }

        stage('Verify JaCoCo Agent') {
            steps {
                script {
                    // Verify JaCoCo agent is running correctly (e.g., by checking the presence of the .exec file)
                    def checkFileCommand = 'docker exec <container_name> [ -f /root/jacoco_agent/jacoco-application.exec ] && echo true || echo false'
                    def fileExists = sh(script: checkFileCommand, returnStdout: true).trim() == 'true'

                    if (!fileExists) {
                        error("JaCoCo exec file not found in the container.")
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleanup or additional steps after the pipeline
        }
    }
}
