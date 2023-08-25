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
        NEXUSIP='44.202.88.40' 
        NEXUSPORT='8081'
        NEXUS_GRP_REPO='vpro-maven-group'
        NEXUS_LOGIN='nexuslogin'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.202.88.40:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPOGRP_ID    = "vprofile-grp-repo"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
    }


    stages {
        stage ('Build') {
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }
    }
}