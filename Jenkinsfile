pipeline {
    agent any

    stages {
        stage('Clean Reports') {
            steps {
                sh '''
                echo '********* Cleaning Workspace Stage Started **********'
                rm -rf test-reports/
                echo '********* Cleaning Workspace Stage Finished **********'
                '''
            }
        }

        stage('Build Stage') {
            steps {
                sh '''
                echo '********* Build Stage Started **********'
                pip3 install -r requirements.txt
                pyinstaller --onefile app.py
                echo '********* Build Stage Finished **********'
                '''
            }
        }

        stage('Testing Stage') {
            steps {
                sh '''
                echo '********* Test Stage Started **********'
                python3 test.py
                echo '********* Test Stage Finished **********'
                '''
            }
        }

        stage('Configure Artifactory') {
            steps {
                script {
                    echo '********* Configure Artifactory Started **********'
                    def userInput = input(
                        id: 'userInput',
                        message: 'Enter password for Artifactory',
                        parameters: [
                            [$class: 'TextParameterDefinition', defaultValue: 'password', description: 'Artifactory Password', name: 'password']
                        ]
                    )
                    sh """
                    jfrog rt c artifactory-demo --url=http://34.68.191.118:8081/artifactory --user=admin --password=${userInput}
                    """
                    echo '********* Configure Artifactory Finished **********'
                }
            }
        }

        stage('Sanity Check') {
            steps {
                input "Does the staging environment look ok?"
            }
        }

        stage('Deployment Stage') {
            steps {
                input "Do you want to Deploy the application?"
                echo '********* Deploy Stage Started **********'
                timeout(time: 1, unit: 'MINUTES') {
                    sh 'python3 app.py'
                }
                echo '********* Deploy Stage Finished **********'
            }
        }
    } // closes stages

    post {
        always {
            echo 'We came to an end!'
            archiveArtifacts artifacts: 'dist/*.exe', fingerprint: true
            junit 'test-reports/*.xml'
            script {
                if (currentBuild.currentResult == 'SUCCESS') {
                    echo '********* Uploading to Artifactory is Started **********'
                    //sh 'jfrog rt u "dist/*.exe" generic-local'
                    //sh 'Powershell.exe -executionpolicy remotesigned -File build_script.ps1'
                    echo '********* Uploading Finished **********'
                }
            }
            deleteDir()
        }

        success {
            echo 'Build Successful!!'
        }

        failure {
            echo 'Sorry mate! Build Failed :('
        }

        unstable {
            echo 'Run was marked as unstable'
        }

        changed {
            echo 'Pipeline state has changed.'
        }
    } // closes post
} // closes pipeline
