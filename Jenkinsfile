def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        //get the private ip not the public ip , they are in the same network they talk with each other with private ip
        SNAP_REPO='vprofile-snapshot'
        NEXUS_USER='admin'
        NEXUS_PASS='admin123'
        RELEASE_REPO='vprofile-release'
        CENTRAL_REPO='vpro-maven-central'
        NEXUSIP='172.31.89.35' 
        NEXUSPORT='8081'
        NEXUS_GRP_REPO='vpro-maven-group'
        NEXUS_LOGIN='nexuslogin'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.202.88.40:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPOGRP_ID    = "vprofile-grp-repo"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        SONARSERVER ='sonarserver'
        SONARSCANNER='sonarscanner'
        NEXUSPASS= credentials('nexuspass')
        
    }


    stages {
        stage ('Build') {
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
        
            post {
                success {
                    echo "Now archiving ."
                    archiveArtifacts artifacts:'**/*.war'
                }
            }
        }
        
        stage ('Test'){
            steps{
                //unit test
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle Analysis'){
            steps{
                //code analysis
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis'){
             environment {
                scannerHome = tool "${SONARSCANNER}"
                SONAR_TOKEN = credentials('sonartoken')
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.login=${SONAR_TOKEN} \
                -Dsonar.projectKey=vprofile \
               -Dsonar.projectName=vprofile-repo \
               -Dsonar.projectVersion=1.0 \
               -Dsonar.sources=src/ \
               -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
               -Dsonar.junit.reportsPath=target/surefire-reports/ \
               -Dsonar.jacoco.reportsPath=target/jacoco.exec \
               -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate'){
            steps{
                //waiting for Quality gate to respond for 10 min else abortPipeline
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
            }
            }
        }
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                    //informations about NEXUS :

                    nexusVersion: 'nexus3',//the nexus version used
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",//nexus url
                    groupId: 'QA',//will create a folder with 'com.example' name and put our artifact inside that
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}", // versionning ( build_id will change and build_stamp will change also => good combination for versionning)
                    repository: "${RELEASE_REPO}",//repos where we want to upload
                    credentialsId: "${NEXUS_LOGIN}",//nexus credentials to gain access 
                    
                    //informations about Our ARTIFACT :
                    
                    artifacts: [
                        [artifactId: 'vproapp',//this will be the prefix for our artifact => so the artifact name will be : vproapp-BUILD_ID-BUILD_TIMESTAMP.war
                         classifier: '',
                         file: 'target/vprofile-v2.war', // the artifact that we want to upload
                         type: 'war']
                    ]
                )
            }
        }
        stage('Ansible Deploy to staging'){
            // stage to execute our ansible playbooks by using Ansible plugin.
            steps{
                ansiblePlaybook([
                    playbook: 'ansible/site.yml',
                    inventory: 'ansible/stage.inventory',
                    installation: 'ansible',
                    colorized: true,
                    credentialsId: 'applogin',
                    //disableHostKeyChecking: true otherwise playbook execution will fail. It will say host key checking is enabled so make sure this option is there extra variables. 
                    disableHostKeyChecking: true,
                    extraVars   : [
                        USER: "admin",
                        PASS: "${NEXUSPASS}"
                        nexusip: "172.31.89.35",
                        reponame: "vprofile-release",
                        groupid: "QA",
                        time: "${env.BUILD_TIMESTAMP}",
                        build: "${env.BUILD_ID}",
                        artifactid: "vproapp",
                        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"

                    ]
                ])
            }

        }
    }
    post {
        //always block means it will always get executed.
        always {
            echo 'Slack Notifications.'
            // "slackSend" is the plugin that we Installed and we are passing 3 arguments to it
            // 3 arguments : one is the channel , one is the color and one is the message 
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
