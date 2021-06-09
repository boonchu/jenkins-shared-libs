#!/usr/bin/groovy

// https://www.jenkins.io/doc/book/pipeline/shared-libraries/
@Library('jenkins-shared-lib@1.0.0') _

pipeline {

    environment{
        def CONFIG_FILE_UUID = '8ac4e324-359d-4b24-9cc3-04893a7d56ce'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Git branch to use for the build.')
        choice(name: 'MAVEN_IMAGE_VERSION', choices: 'maven:3.8.1-openjdk-8\nmaven:3.8.1-openjdk-11\n', description: 'Use Maven Image Version?')
    }

    agent {
        kubernetes {
            defaultContainer 'maven'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: "${params.MAVEN_IMAGE_VERSION}"
    command:
    - sleep
    args:
    - infinity
"""
        }
    }

    options {
        // using the Timestamper plugin we can add timestamps to the console log
        timestamps()
        // ansiColor('xterm')
        // echo '\033[34mHello\033[0m \033[33mcolorful\033[0m \033[35mworld!\033[0m'
        timeout(time: 150, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {
        stage("Initialize Environment") {
            steps {
                script {
                    def rootDir = pwd()
                    echo "LOG-->INFO-->Current Working Directory : ${rootDir}"
                    echo "LOG-->INFO-->BRANCH : ${params.BRANCH}"
                    echo "LOG-->INFO-->MAVEN_IMAGE_VERSION : ${params.MAVEN_IMAGE_VERSION}"
                    echo "LOG-->INFO-->CONFIG_FILE_UUID : ${CONFIG_FILE_UUID}"
                }
                container("maven") {
                    sh """
			env | sort
			mvn -v
			java -version
		    """
                }
                checkout([$class: 'GitSCM', 
                    branches: [[name: "*/${params.BRANCH}"]], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'CleanCheckout']], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'github-credential', url: 'https://github.com/boonchu/spring-boot-reactive-jenkins-pipeline.git']]
                ])
                helloWorld(name:"Darin", dayOfWeek:"Wednesday")
            }
        }
        stage("Maven Build") {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    ARTIFACT_VERSION = pom.version
                    echo "LOG-->INFO-->ARTIFACT_VERSION : ${ARTIFACT_VERSION}"
                }
                container("maven") {
                    // https://shekhargulati.com/2019/02/09/using-jenkins-config-file-provider-plugin-to-allow-jenkins-slave-to-access-mavens-global-settings-xml/
                    configFileProvider([configFile(fileId: "${CONFIG_FILE_UUID}", variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                        sh """
			   echo "artifact version: ${ARTIFACT_VERSION}"
                           mvn clean install -DskipTests=true -f pom.xml -gs $MAVEN_GLOBAL_SETTINGS
			"""
                    }
                }
                script {
                    try {
                           sh 'sudo echo "hello world"'
                    } catch (err) {
                        echo err.getMessage()
                        echo "Error detected, but we will continue."
                    } 
                }         
            } 
        }
        stage("Maven functional Test"){
            steps{
                helloWorldSimple("Boonchu", "Monday")
                configFileProvider([configFile(fileId: '8ac4e324-359d-4b24-9cc3-04893a7d56ce', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                    sh """
                       mvn test -f pom.xml -gs $MAVEN_GLOBAL_SETTINGS
		    """
                }
            }
        }
        stage("Jacoco Code Coverage"){
            steps{
               junit '*/build/test-results/*.xml'
               jacoco(
                   execPattern: '**/path_to_file/jacoco.exec',
                   classPattern: '**/coverage/**',
                   sourcePattern: '**/coverage/**',
                   inclusionPattern: '**/*.class'
	       )
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
