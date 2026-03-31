pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-pkumarcoverity')
        DETECT_EXCLUDED_DETECTOR_TYPES = "GIT"
        BRIDGE_POLARIS_SERVERURL = 'https://poc.polaris.blackduck.com'
        BRIDGE_POLARIS_ACCESSTOKEN = credentials('polaris-se')
    }

    tools {
        maven 'maven-3'
        jdk 'openjdk-21'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    // Get repo name from Git URL
                    def gitUrl = sh(script: "git config --get remote.origin.url", returnStdout: true).trim()
                    env.REPO_NAME = gitUrl.tokenize('/.git')[-1]

                    // Get current branch name
                    env.BRANCH_NAME = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()

                    echo "Repo Name: ${env.REPO_NAME}"
                    echo "Branch Name: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B package'
            }
        }

        stage('Coverity Scan') {
            steps {
                security_scan product: 'polaris',
                    polaris_assessment_types: 'SAST,SCA',
                    polaris_application_name: "pkumarb-cicd",
                    polaris_project_name: "$REPO_NAME",
                    polaris_prComment_enabled: false,
                    polaris_reports_sarif_create: false,
                    coverity_build_command: 'mvn -B -DskipTests package',
                    coverity_clean_command: 'mvn -B clean',
                    github_token: "$GITHUB_TOKEN",
                    include_diagnostics: false,
                    mark_build_status: 'UNSTABLE'
            }
        }
    }

    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: '.bridge/bridge.log, .bridge/*/idir/build-log.txt'
            cleanWs()
        }
    }
}
