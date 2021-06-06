#!/usr/bin/groovy

@Library('github.com/darinpope/github-api-global-lib@master') _

pipeline{
    agent{
        label "node"
    }
    stages{
        stage("A"){
            steps{
                echo "========executing A========"
                try {
                    sleep 5
                    helloWorld("Darin")
                } catch (Exception ex) {
                    println ex.printStackTrace()
                }
                echo "Finishing"
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