#!/usr/bin/env groovy

pipeline {
    //Dont change this. This agent definition keeps the jobs using up nodes for management.
    agent any

    //defining common variables.
    environment {
        JOB_OWNER_CHANNEL = "ae-alert"
        REPO_NAME = "fr-analytics-engines-infra"
    }

    stages {
        stage ('Initialize') {
            steps {
                cleanWs()
                checkout scm: [
                    $class: 'GitSCM', branches: [[name: "develop-nttd-yamada-applytest"]],
                    userRemoteConfigs: [[credentialsId: 'analytics-engines-deployment-account', url: "https://github.com/fastretailing/fr-analytics-engines-infra"]]
                ]
            }
        }

        stage('prepare') {
            options {
                timeout(time: 3, unit: 'MINUTES') 
            }

            steps {
                script {
                    algoList = sh(
                         script: "find . -maxdepth 1 -type d|sed -e '/\\.git/d' -e 's#./##g'"
                        ,returnStdout: true
                    )

                    algo = input(
                        id: 'algoInput',
                        message: '対象アルゴリズムを選択して下さい',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition',
                                name: 'directoryPath',
                                choices: algoList,
                                description: 'select deploy target']
                    ])

                    envList = sh(
                         script: "cd ${algo}/config/fr;find . -maxdepth 1 -type d|sed -e 's#./##g'"
                        ,returnStdout: true
                    )

                    algoenv = input(
                        id: 'envInput',
                        message: '実行環境を選択して下さい',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition',
                                name: 'algoEnvPath',
                                choices: envList,
                                description: 'select deploy environment']
                    ])
                }
            }
        }
        stage('confirm changes') {
            options {
                timeout(time: 3, unit: 'MINUTES') 
            }

            steps {
                dir("./${algo}") {
                    sh "pwd"
                    sh "ls -la"
                    //sh "terraform init"
                    //sh "terraform plan -var-file=config/fr/${algoenv}/ap-northeast-1.tfvars"
                }
            }
        }
        stage('action') {

            steps {
                input(
                    id: 'finalconfirm',
                    message: 'かくにんしましょう',
                    ok: 'かくにんしたのでじっこうします'
                )
            }
        }
    }
    post{
        always {
            cleanWs()
        }
        // 成功した時
        success {
            slackSend (
                tokenCredentialId: 'analytics-engines-jenkins',
                baseUrl: "https://fastretailing.slack.com/services/hooks/jenkins-ci/",
                channel: "${JOB_OWNER_CHANNEL}",
                color: "#00FF00",
                message: "The pipeline ${currentBuild.fullDisplayName} completed successfully.")
        }
        // 失敗した時
        failure {
            slackSend (
                tokenCredentialId: 'analytics-engines-jenkins',
                baseUrl: "https://fastretailing.slack.com/services/hooks/jenkins-ci/",
                channel: "${JOB_OWNER_CHANNEL}",
                color: "#FF0000",
                message: "The pipeline ${currentBuild.fullDisplayName} is failure.")
        }
        // 中止した時
        aborted {
            slackSend (
                tokenCredentialId: 'analytics-engines-jenkins',
                baseUrl: "https://fastretailing.slack.com/services/hooks/jenkins-ci/",
                channel: "${JOB_OWNER_CHANNEL}",
                color: "#FF0000",
                message: "The pipeline ${currentBuild.fullDisplayName} is aborted.")
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
}
