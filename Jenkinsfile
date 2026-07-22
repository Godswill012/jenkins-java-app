#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@starting-code', retriever: modernSCM(
    [
        $class: 'GitSCMSource',
        remote: 'https://github.com/Godswill012/jenkins-shared-library.git',
        credentialsId: 'github-credentials'
    ]
)

def gv

pipeline {
    agent any

    tools {
        maven 'maven-3.9'
    }

    environment {
        IMAGE_NAME = "godswill012/demo-app:jma-${BUILD_NUMBER}"
    }

    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }

        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage("build and push image") {
            steps {
                script {
                    buildImage()
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }
    }
}


/*
#!/usr/bin/env groovy
library identifier: 'jenkins-shared-library@starting-code', retriever: modernSCM(
        [$class: 'GitSCMSource',
        remote: 'https://github.com/Godswill012/jenkins-shared-library.git',
        credentialsId: 'github-credentials'])

def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }

        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage("build and push image") {
            steps {
                script {
                    buildImage 'godswill012/demo-app:jma-3.0'
                    dockerLogin()
                    dockerPush 'godswill012/demo-app:jma-3.0'
                }
            }
        }
        
        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }               
    }
}
*/



