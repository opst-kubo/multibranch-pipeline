#!/usr/bin/env groovy

pipeline {
    //Dont change this. This agent definition keeps the jobs using up nodes for management.
    agent any

    //defining common variables.
    environment {
        JOB_OWNER_CHANNEL = "alert-analytics-eng"

        REPO_NAME = "fr-analytics-engines-infra"
    }

    //parameters.
    parameters {
        choice(name: 'TARGET_ALGO',
               choices: ['', 'salesmodel', 'colorsize', 'invopt'],
               description: 'Select target') 
    }

    stages {
        stage ('Initialize') {
            steps {
                cleanWs()
            }
        }

        stage('Confirme') {
            steps {
                checkout scm: [
                    $class: 'GitSCM', branches: [[name: GIT_COMMIT]],
                    userRemoteConfigs: [[credentialsId: 'analytics-engines-deployment-account', url: GIT_URL]]
                ]

                sh """
                    set -xe

                    mkdir -p ./"${params.TARGET_ALGO}"
                    cd ./"${params.TARGET_ALGO}"
                    echo "terraform apply!!!!"

                """
            }
        }
        stage('Execute') {
            options {
                timeout(time: 3, unit: 'MINUTES') 
            }
            when {
                branch 'master'
            }
            steps {
                checkout scm: [
                    $class: 'GitSCM', branches: [[name: GIT_COMMIT]],
                    userRemoteConfigs: [[credentialsId: 'analytics-engines-deployment-account', url: GIT_URL]]
                ]

                sh """
                    set -xe

                    mkdir -p ./"${params.TARGET_ALGO}"
                    cd ./"${params.TARGET_ALGO}"
                    echo "terraform deploy!!!!"

                """
            }
        }
    }
    post{
        // 成功した時
        success {
            sendToSlack 'SUCCESS', JOB_OWNER_CHANNEL, "Job result"
        }
        // 失敗した時
        failure {
            sendToSlack 'FAILURE', JOB_OWNER_CHANNEL, "Job result"
        }
        // 不安定な時
        unstable {
            sendToSlack 'UNSTABLE', JOB_OWNER_CHANNEL, "Job result"
        }
        // 中止した時
        aborted {
            sendToSlack 'ABORTED', JOB_OWNER_CHANNEL, "Job result"
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
}

def getSafeStringOfBranchName(String branch_name){
    return branch_name.replaceAll("/","__")
}

