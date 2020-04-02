#!groovy
import groovy.json.JsonSlurperClassic
node {
    //default variables 
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    //sfdx executable 
    def toolbelt = tool 'toolbelt'
    def SFDC_USERNAME

    //production variables 
    def HUB_ORG=env.HUB_ORG_DEVOPS_PRD
    def SFDC_HOST = env.SFDC_HOST_DEVOPS_PRD
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_PRD
    def ORG_ALIAS_DEVOPS_PRD = env.ORG_ALIAS_DEVOPS_PRD

    //sandbox variables or other environments 
    
    

    
    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {

            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --setalias ${ORG_ALIAS_DEVOPS_PRD} --instanceurl ${SFDC_HOST} --jwtkeyfile \"${jwt_key_file}\" "
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --targetusername ${ORG_ALIAS_DEVOPS_PRD}"
            printf rmsg
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null

        }

        stage('Push To Test Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
            // assign permset
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
            if (rc != 0) {
                error 'permset:assign failed'
            }
        }

        stage('Run Apex Test') {
            bat "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }

        stage('Delete Test Org') {

        timeout(time: 120, unit: 'SECONDS') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:delete --targetusername ${SFDC_USERNAME} --noprompt"
            if (rc != 0) {
                error 'org deletion request failed'
            }
        }
    }
    }
}

// retorna o comanda para o SO do Jenkins 
def getCommand(){
  if (isUnix()) {
    return "sh"
  }else{
    return "bat"
  }
}