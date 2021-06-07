#!/usr/bin/groovy

// https://www.jenkins.io/doc/book/pipeline/shared-libraries/
@Library('jenkins-shared-lib@1.0.0') _

pipeline{
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
    stages{
        stage("A"){
            steps{
                helloWorld(name:"Darin", dayOfWeek:"Wednesday")
            }
            post{
                always{
                    echo "========always========"
                }
                success{
                    echo "========A executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }
        }
        stage("B"){
            steps{
                helloWorldExternal(name:"Darin", dayOfWeek:"Wednesday")
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
