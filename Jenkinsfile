pipeline {
    agent any
    tools {
        jdk "${JDK_TOOL}"
        snyk "${SNYK_TOOL}"
    }
    stages {
        stage('Clone repo') {
            steps {
                git "${REPO}"
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                dir('Android/MSTG-Android-Java-App') {
                    sh './gradlew build'
                }

            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
                dir('Android/MSTG-Android-Java-App') {
                    snykSecurity failOnIssues: false, snykInstallation: "${SNYK_TOOL}", snykTokenId: "${SNYK_TOKEN_ID}"
                    withCredentials([string(credentialsId: "${MOBSF_TOKEN_ID}", variable: "MOBSF_TOKEN")]){
                        sh '''
                            hash=$(curl -F 'file=@app/build/outputs/apk/release/app-release-unsigned.apk' http://${MOBSF_HOST}/api/v1/upload -H "Authorization:${MOBSF_TOKEN}" | grep -o -e "[0-9a-f]\\{32\\}")
                            curl -X POST --url http://${MOBSF_HOST}/api/v1/scan --data "hash=${hash}" -H "Authorization:${MOBSF_TOKEN}"
                            curl -X POST --url http://${MOBSF_HOST}/api/download_pdf --data "hash=${hash}" -H "Authorization:${MOBSF_TOKEN}" -o mobsf_report_${BUILD_NUMBER}.pdf
                        '''
                        
                    }
                    archiveArtifacts "mobsf_report_${BUILD_NUMBER}.pdf"
                    withCredentials([usernamePassword(credentialsId: "${ARTIFACTORY_TOKEN_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'curl -u${USERNAME}:${PASSWORD} -T $JENKINS_HOME/jobs/jobs/${JOB_NAME}/builds/${BUILD_NUMBER}/archive/*.html "${ARTIFACTORY_URL}/artifactory/example-repo-local/${JOB_NAME}/snyk_report_${BUILD_NUMBER}.html"'
                        sh 'curl -u${USERNAME}:${PASSWORD} -T $JENKINS_HOME/jobs/jobs/${JOB_NAME}/builds/${BUILD_NUMBER}/archive/mobsf_report_${BUILD_NUMBER}.pdf "${ARTIFACTORY_URL}/artifactory/example-repo-local/${JOB_NAME}/mobsf_report_${BUILD_NUMBER}.pdf"'
                    }
                    
                }
                dir('Android/MSTG-Android-Java-App/'){
                    
                }
            }
        }
    }
}