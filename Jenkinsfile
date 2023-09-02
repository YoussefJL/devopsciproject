def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any

    environment {
        //this is nexus pass that will be used by Ansible playbook to log into Nexus server
        NEXUSPASS= credentials('nexuspass')
        
    }


    stages {
        
        //this stage is to ask user for TIME & BUILD variables values to enter , we need those in the next stage
        stage('Setup parameters') {
            steps {
                script { 
                    properties([
                        parameters([
                            string(
                                defaultValue: '', 
                                name: 'BUILD', 
                            ),
							string(
                                defaultValue: '', 
                                name: 'TIME', 
                            )
                        ])
                    ])
                }
            }

       
        stage('Ansible Deploy to Prod'){
            // stage to execute our ansible playbooks by using Ansible plugin.
            steps{
                ansiblePlaybook([
                    playbook: 'ansible/site.yml',
                    inventory: 'ansible/prod.inventory',
                    installation: 'ansible',
                    colorized: true,
                    credentialsId: 'applogin-prod',
                    //disableHostKeyChecking: true otherwise playbook execution will fail. It will say host key checking is enabled so make sure this option is there extra variables. 
                    disableHostKeyChecking: true,
                    extraVars   : [
                        USER: "admin",
                        PASS: "${NEXUSPASS}",
                        nexusip: "172.31.89.35",
                        reponame: "vprofile-release",
                        groupid: "QA",
                        // TIME & BUILD variables which we will take as an input from the user build, this is our own created variable.
                        // now these variables does not exist yet, we r going to take input from the user.
                        time: "${env.TIME}",
                        build: "${env.BUILD}",
                        artifactid: "vproapp",
                        vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"

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
}