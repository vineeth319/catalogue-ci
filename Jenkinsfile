pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        packageVersion = ''
        nexusURL = '98.92.51.222:8081'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {

        stage('Get the Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    packageVersion = packageJson.version

                    echo "application version: ${packageVersion}"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x "*.zip"
                    ls -ltr
                '''
            }
        }

        stage('Publish Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: nexusURL,
                    groupId: 'com.roboshop',
                    version: packageVersion,
                    repository: 'catalogue',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [
                            artifactId: 'catalogue',
                            classifier: '',
                            file: 'catalogue.zip',
                            type: 'zip'
                        ]
                    ]
                )
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def params = [
                        string(name: 'version', value: packageVersion),
                        string(name: 'environment', value: 'dev')
                    ]

                    build job: 'catalogue-deploy',
                          wait: true,
                          parameters: params
                }
            }
        }
    }

    post {
        always {
            echo 'Workspace cleanup'
            deleteDir()
        }

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}