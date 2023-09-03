def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

//this command is going to fetch the last successful build ID from cicd-jenkins-bean-stage and store it in the variable 
def buildNumber = Jenkins.instance.getItem('cicd-jenkins-bean-stage').lastSuccessfulBuild.number


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
        ARTIFACT_NAME= "vprofile-v${buildNumber}.war"
        AWS_S3_BUCKET='youssefvprocicdbean'
        AWS_EB_APP_NAME='youssefvproapp'
        AWS_EB_ENVIRONMENT='Youssefvproapp-prod-env'
        AWS_EB_APP_VERSION="${buildNumber}"

        
        
    }


    stages {
        stage("Deploy to Stage Bean"){
            steps{
                //here we will be executing all AWS CLI commands
                // those commands need AWS Credentials 
                withAWS(credentials: 'awscreds', region: 'us-east-1'){
                   sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
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
