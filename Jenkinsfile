def COLOR_MAP = [
    SUCCESS: 'good', 
    FAILURE: 'danger',
]

pipeline {
    agent any

    environment {
        SONAR_URL = "http://192.168.33.10:9000"
        JAVA_HOME = tool('JDK')  
    }

    tools {
        nodejs "NODE" 
        jdk "JDK"
        maven "MVN"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Cache npm Dependencies') {
            steps {
                script {
                    if (fileExists('GitHubDir')) {
                        echo 'Restoring npm dependencies from cache...'
                        unstash name: 'GitHubDir'
                    } else {
                        echo 'No node_modules found in cache.'
                    }
                }
            }
        }

        stage('Fetching the Code') {
            steps {
                script {
                    // Use the current branch as part of the clone URL
                    def scmBranch = env.BRANCH_NAME ?: 'master'
                    sh "git clone -b ${scmBranch} https://github.com/DevOpsByOmer/GitHub-.git"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'cd GitHub- && npm install'
                }
            }
        }
         stage('Sonar Code Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar/bin/sonar-scanner -X -Dsonar.projectKey=reactapp \
                        -Dsonar.projectName=reactapp \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=/var/lib/jenkins/workspace/React_project/GitHub-/src/'''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'cd GitHub- && npm run build'
                }
            }
        }

        stage('Stash npm Dependencies') {
            steps {
                script {
                    echo 'Stashing npm dependencies for future builds...'
                    stash name: 'GitHubDir', includes: '/var/lib/jenkins/workspace/Reacr_project/GitHub-/node_modules/**', allowEmpty: true
                }
            }
        }

        stage('Deploy to Nginx') {
            steps {
                script {
                    echo 'Deploying to Nginx...'
                    sh 'rsync -a GitHub-/build/ /var/www/html/'
                }
            }

    post {
        always {
            echo 'Slack Notification.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n",
                botUser: false,
                tokenCredentialId: 'slacktoken',
                notifyCommitters: false
        }
    }
}
}
}
