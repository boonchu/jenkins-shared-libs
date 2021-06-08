#!/usr/bin/groovy

// https://www.jenkins.io/doc/book/pipeline/shared-libraries/
@Library('jenkins-shared-lib@1.0.0') _

pipeline {

    environment{
        def EARLIESTDATE = '0'
        def LATESTDATE = '0'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Git branch to use for the build.')
        choice(name: 'APPEND_COMMIT_VERSION', choices: 'ON\nOFF', description: 'Append Git commit ID to build version?')
        choice(name: 'DEBUG_SQL', choices: '0\n1\n2', description: 'Level of SQL tracing.\n0 - disabled.')
        choice(name: 'SAP_CONNECTION_POOL_DEBUG', choices: '0\n1', description: 'Level of SAP connection pool tracing.\nWorks only in debug builds.\n0 - disabled.')
        choice(name: 'BUILD_TYPE', choices: 'RelWithDebInfo\nDebug\nRelease', description: 'Build type')
    }

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: adoptopenjdk/openjdk11:alpine
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }

    options {
        // using the Timestamper plugin we can add timestamps to the console log
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {
        stage("Build") {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/master']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'CleanCheckout']], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'github-credential', url: 'https://github.com/boonchu/spring-boot-reactive-jenkins-pipeline.git']]
                ])
                helloWorld(name:"Darin", dayOfWeek:"Wednesday")
                script {
                    try {
                        sh "sudo docker rmi frontend-test"
                    } catch (err) {
                        echo err.getMessage()
                        echo "Error detected, but we will continue."
                    } 
                }
            } 
        }
        stage("Test"){
            steps{
                helloWorldSimple("Boonchu", "Monday")
                script {
                    def rootDir = pwd()
                    echo "LOG-->INFO-->Current Working Directory : ${rootDir}"
                    echo "LOG-->INFO-->BRANCH : ${params.BRANCH}"
                    echo "LOG-->INFO-->DEBUG_SQL : ${params.DEBUG_SQL}"
                    def actions = {
                        put("BRANCH", "master")
                        put("DEBUG_SQL", "1")
                    }
                    fooProject(actions)
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
