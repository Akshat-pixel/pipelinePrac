pipeline {
    agent any
    tools {
        maven "mvn"
        jdk "jdk17"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.30.191:8081"
        NEXUS_REPOSITORY = "vprofile"
        NEXUS_GROUPID = "QA"
        NEXUS_CREDENTIAL_ID = "nexuscred"
        NEXUS_ARTIFACT_ID = "vproapp"
        ARTVERSION = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
        scannerHome = tool 'sonar6.2'

    }
    stages {
        stage('Fetch Code'){
            steps{
                git branch: 'main', url: 'https://github.com/Akshat-pixel/pipelinePrac.git'
            }
        }
        stage('Build'){
            steps{
                sh 'mvn install -Dskiptest'
            }
            post {
                success{
                    echo "Archiving Artifact"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Checkstyle analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage("Sonar Code Analysis"){
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportsPath=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1,unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
                    nexusUrl: NEXUS_URL,
                    groupId: NEXUS_GROUPID,
                    version: ARTVERSION,
                    repository: NEXUS_REPOSITORY,
                    credentialsId: NEXUS_CREDENTIAL_ID,
                    artifacts: [
                        [artifactId: NEXUS_ARTIFACT_ID,
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']])
            }
        }
        stage('Trigger Ansible Deployment') {
            steps {
                sshagent(credentials: ['ansible-kp']) {
                    sh """
                    ssh -t -o StrictHostKeyChecking=no ubuntu@54.234.125.118 << EOF
                    ansible-playbook /home/ubuntu/docker_install/docker_setup.yaml 
                    ansible-playbook tomcat_setup.yaml --extra-vars "artifact_url=http://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/${NEXUS_GROUPID}/${NEXUS_ARTIFACT_ID}/${ARTVERSION}/${NEXUS_ARTIFACT_ID}-${ARTVERSION}.war"
                    << EOF
                    """
                }
            }
        }

    }
}
