#!/usr/bin/groovy

// https://www.jenkins.io/doc/book/pipeline/shared-libraries/
@Library('jenkins-shared-lib@1.0.0') _

pipeline {

    parameters {
        string(name: 'PARAM1', defaultValue: '', description: 'Param 1?')
        string(name: 'PARAM2', defaultValue: '', description: 'Param 2?')
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

    // using the Timestamper plugin we can add timestamps to the console log
    options {
        timestamps()
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
                    def config = {
                        put("param1", [string(name: 'PARAM1', value: params.PARAM1)])
                        put("param2", [string(name: 'PARAM2', value: params.PARAM2)])
                    }
                    fooProject(config)
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
