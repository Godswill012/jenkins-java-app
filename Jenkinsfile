def gv

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t godswill012/demo-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push godswill012/demo-app:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image...'
                }
            }
        }
        stage('commit version update'){
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                        sh 'git config --global user.email "josephnsudegodswill@gmail.com"'
                        sh 'git config --global user.name "Godswill012"'

                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'

                        sh 'git add pom.xml'
                        sh 'git commit -m "ci: version bump"'
                                                sh '''set +x
                            helper="$(mktemp)"
                            trap 'rm -f "$helper"' EXIT
                            cat > "$helper" <<'EOF'
#!/bin/sh
case "$1" in
    *Username*) printf '%s' "$GIT_USERNAME" ;;
    *Password*) printf '%s' "$GIT_PASSWORD" ;;
esac
EOF
                            chmod 700 "$helper"
                            GIT_ASKPASS="$helper" GIT_TERMINAL_PROMPT=0 \
                                                            git push https://github.com/Godswill012/jenkins-java-app.git HEAD:jenkins-jobs
                            rm -f "$helper"'''
                    }
                }
            }
         }
    }
}
