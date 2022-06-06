pipeline {

    agent { dockerfile true }

    environment {
        CREDS = credentials('amt-tes-userpass')    // This provides 3 variables: CREDS=username:password, CREDS_USR=username, CREDS_PSW=password (https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#usernames-and-passwords)
        WEB_HOOK_URL = credentials('amt-tes-webhook')   
        GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
        ENV = ['stage','dev', 'prod']
    }


    triggers {
         cron('H 08 * * *')
    }

    tools {nodejs "nodejs"}

    parameters {
        choice(name: 'BROWSER', choices: ['chrome', 'electron', 'firefox'], description: 'Pick the web browser you want to use to run your scripts')
        choice(name: 'ENVIRONMENT', choices: ${ENV}, description: 'Pick the environment to test against')
        choice(name: 'TEST', choices: ['@regression','@smoke'], description: 'Pick the type of test to runned')
        string(name: 'TAG', defaultValue: '', description: 'Run collection of test E.g: @navigation')
        password(name: 'USERNAME', defaultValue: 'automation_teacher')
        password(name: 'PASSWORD', defaultValue: 'Test1234!')
    }

    options {
        ansiColor('xterm')
    }
    
    stages {
        
        stage('Verify'){
            steps {
                sh 'npm i'
            }
        }
        stage('Run Tests') {
            parallel {
                stage('Test Home') {
                    steps {
                        script {
                            if ( currentBuild.rawBuild.getCauses()[0].toString().contains('UserIdCause') ){
                                if (TAG?.isEmpty()) {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=${ENVIRONMENT} --spec 'cypress/integration/Home/*.feature' TAGS='${TEST}'"
                                } else {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=${ENVIRONMENT} --spec 'cypress/integration/Home/*.feature' TAGS='${TAG}'"
                                }
                            } else {
                                if(JOB_NAME == 'amt-tes-prod'){
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=prod --spec 'cypress/integration/Home/*.feature' TAGS='@regression'"
                                } else {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=stage --spec 'cypress/integration/Home/*.feature' TAGS='@regression'"
                                }
                            }
                        } 
                    }
                }
                stage('Test Rooms') {
                    steps {
                        script {
                            if ( currentBuild.rawBuild.getCauses()[0].toString().contains('UserIdCause') ){
                                if (TAG?.isEmpty()) {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=${ENVIRONMENT} --spec 'cypress/integration/Rooms/*.feature' TAGS='${TEST}'"
                                } else {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=${ENVIRONMENT} --spec 'cypress/integration/Rooms/*.feature' TAGS='${TAG}'"
                                }
                            } else {
                                if(JOB_NAME == 'amt-tes-prod'){
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=prod --spec 'cypress/integration/Rooms/*.feature' TAGS='@regression'"
                                } else {
                                    sh "npx cypress-tags run --browser ${BROWSER} --env configFile=stage --spec 'cypress/integration/Rooms/*.feature' TAGS='@regression'"
                                }
                            }
                
                        } 
                    }
                }
            }
        }
        

    }
    post {
        always {
           sh 'node cucumber-html-report.js'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'cypress/reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
        }
    } 
}