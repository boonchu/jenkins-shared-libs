#!/usr/bin/groovy

// https://www.jenkins.io/doc/book/pipeline/shared-libraries/
@Library('jenkins-shared-lib@1.0.0') _

pipeline {
    environment {
        def CONFIG_FILE_UUID   = '8ac4e324-359d-4b24-9cc3-04893a7d56ce'
        def SONAR_SERVER_URL   = 'http://172.30.30.102:9000'
        def SONAR_SCANNER_HASH = '6d401f63ef2d3cbae6c1536064077d2178bb6d2e'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Git branch to use for the build.')
        choice(name: 'MAVEN_IMAGE_VERSION', choices: 'maven:3.8.1-openjdk-11\nmaven:3.8.1-openjdk-8\n', description: 'Choice of Maven Image Versions?')
        choice(name: 'SONAR_PLUGIN_VERSION', choices: '3.9.0.2155\n3.8.0.2131\n3.7.0.1746\n', description: 'Choice of Sonar Plugin Versions?')
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
        retry(2)
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
                    echo "LOG-->INFO-->SONAR_PLUGIN_VERSION : ${SONAR_PLUGIN_VERSION}"
                    echo "LOG-->INFO-->SONAR_SERVER_URL : ${SONAR_SERVER_URL}"
                    echo "LOG-->INFO-->SONAR_SCANNER_HASH : ${SONAR_SCANNER_HASH}"
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
                container("maven") {
                    configFileProvider([configFile(fileId: "${CONFIG_FILE_UUID}", variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                        sh """
                           mvn test -f pom.xml -gs $MAVEN_GLOBAL_SETTINGS
                        """
                    }
                }
            }
        }
        stage("Static Code Analysis"){
            steps{
               helloWorldSimple("Boonchu", "Tuesday")
               container("maven") {
                   // https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/
                   configFileProvider([configFile(fileId: "${CONFIG_FILE_UUID}", variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                       sh """
		                    mvn clean compile org.sonarsource.scanner.maven:sonar-maven-plugin:${SONAR_PLUGIN_VERSION}:sonar -f pom.xml \
                              -Dsonar.projectKey=maven-code-analysis \
                              -Dsonar.host.url=${SONAR_SERVER_URL} \
                              -Dsonar.login=${SONAR_SCANNER_HASH} \
		                    -gs ${MAVEN_GLOBAL_SETTINGS}
                       """
                   }
               }
            }
        }

        stage("Jacoco Code Coverage"){
            steps{
                helloWorldSimple("Boonchu", "Wednesday")
                container("maven") {
                    configFileProvider([configFile(fileId: "${CONFIG_FILE_UUID}", variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                       withSonarQubeEnv(installationName: 'sonarqube-server', envOnly: true) {
                            sh """
                                echo ${env.SONAR_HOST_URL}
		                        mvn org.jacoco:jacoco-maven-plugin:prepare-agent -f pom.xml clean test \
                                          -Dautoconfig.skip=true -Dmaven.test.skip=false \
                                          -Dmaven.test.failure.ignore=true sonar:sonar \
                                          -Dsonar.host.url=${SONAR_SERVER_URL} \
                                          -Dsonar.login=${SONAR_SCANNER_HASH} \
                                          -gs ${MAVEN_GLOBAL_SETTINGS}
                               """
                            // junit allowEmptyResults: true, testResults: '**/test-results/*.xml'
                        }
                    }

                    sleep(10)

                    // https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/#header-6
                    // use the waitForQualityGate method exposed by the plugin to wait until the SonarQube server 
                    // has called the Jenkins webhook
                    timeout(time: 5, unit: 'MINUTES') {
                        script {
                            // def qg = waitForQualityGate(webhookSecretId: 'sonar-webhook')
                            def qg = waitForQualityGate(abortPipeline: true)
                            if (qg.status != 'OK') {
                               error "Pipeline aborted due to a quality gate failure: ${qg.status}"
                            }
                        }
                    }

                    jacoco(
                        buildOverBuild: false,
                        changeBuildStatus: true,
                        classPattern: '**/classes',
                        execPattern: '**/**.exec',
                        sourcePattern: '**/src/main/java',
                        inclusionPattern: '**/*.java,**/*.groovy,**/*.kt,**/*',
                        minimumMethodCoverage: '10',
                        maximumMethodCoverage: '85',
                        minimumClassCoverage: '10',
                        maximumClassCoverage: '85',
                        minimumLineCoverage: '10',
                        maximumLineCoverage: '85'
	                )
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
            // script {
               // def logContent = Jenkins.getInstance().getItemByFullName(env.JOB_NAME).getBuildByNumber(Integer.parseInt(env.BUILD_NUMBER)).logFile.text
               // writeFile file: "log-build.txt", text: logContent
            // }
        }
    }
}
