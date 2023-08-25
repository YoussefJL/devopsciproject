pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO='vprofile-snapshot'
        NEXUS_USER='admin'
        NEXUS_PASS='admin123'
        RELEASE_REPO='vprofile-release'
        CENTRAL_REPO='vpro-maven-central'
        //get the private ip not the public ip , they are in the same network they talk with each other with private ip
        NEXUSIP='44.202.88.40' 
        NEXUSPORT='8081'
        NEXUS_GRP_REPO='vpro-maven-group'
        NEXUS_LOGIN='nexuslogin'
    }


    stages {
        stage ('Build') {
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }
    }
}