#!groovy
import groovy.json.JsonSlurperClassic
node {
    //default variables 
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    //sfdx executable 
    def toolbelt = tool 'toolbelt'
    def SFDC_USERNAME
    def SFDC_DEVELOPER_MODE = env.SFDC_DEVELOPER_MODE;

    //production variables 
    def HUB_ORG=env.HUB_ORG_DEVOPS_PRD
    def SFDC_HOST = env.SFDC_HOST_DEVOPS_PRD
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_PRD
    def ORG_ALIAS_DEVOPS_PRD = env.ORG_ALIAS_DEVOPS_PRD

    //sandbox variables or other environments 

    if(env.BRANCH_NAME.startsWith('developer_')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_PRD} -p"
                if (rc != 0) { error ' ERROR IN LOGOUT ' }
            echo '------------------ FINALIZANDO O LOGOUT '
        }
    
        stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    deployScratch(CONNECTED_APP_CONSUMER_KEY, HUB_ORG, ORG_ALIAS_DEVOPS_PRD, SFDC_HOST, jwt_key_file)
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg()
                }
                stage('APEX TEST'){
                    runApexTest()
                }
                stage('GET RESULTS'){
                    junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
                }
                stage('DELETE SCRATCH ORG'){
                    echo '------------------ INICIANDO O SCRATCH ORG'
                    runDeleteScrathOrg(SFDC_USERNAME)
                    echo '------------------ FINALIZANDO O SCRATCH ORG'
                }
            }
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }
        

        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }

    }
    if(env.BRANCH_NAME.startsWith('release_')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_PRD} -p"
                if (rc != 0) { error ' ERROR IN LOGOUT ' }
            echo '------------------ FINALIZANDO O LOGOUT '
        }
    
        stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }

        stage('SONAR'){
            echo '------------------ INICIANDO O SONAR'
            
            echo '------------------ FINALIZANDO O SONAR'
        }

        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    }
    if(env.BRANCH_NAME.startsWith('uat')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_PRD} -p"
                if (rc != 0) { error ' ERROR IN LOGOUT ' }
            echo '------------------ FINALIZANDO O LOGOUT '
        }
    
        stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }

        stage('SONAR'){
            echo '------------------ INICIANDO O SONAR'
            
            echo '------------------ FINALIZANDO O SONAR'
        }
        stage('MANUAL STEP'){
            echo '------------------ INICIANDO O MANUAL STEP'
            
            echo '------------------ FINALIZANDO O MANUAL STEP'
        }

        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    }
    if(env.BRANCH_NAME.startsWith('hotfix_')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_PRD} -p"
                if (rc != 0) { error ' ERROR IN LOGOUT ' }
            echo '------------------ FINALIZANDO O LOGOUT '
        }
    
        stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }

        stage('SONAR'){
            echo '------------------ INICIANDO O SONAR'
            
            echo '------------------ FINALIZANDO O SONAR'
        }
        stage('MANUAL STEP'){
            echo '------------------ INICIANDO O MANUAL STEP'
            
            echo '------------------ FINALIZANDO O MANUAL STEP'
        }

        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    }
    if(env.BRANCH_NAME.startsWith('master')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_PRD} -p"
                if (rc != 0) { error ' ERROR IN LOGOUT ' }
            echo '------------------ FINALIZANDO O LOGOUT '
        }
    
        stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }

        stage('SONAR'){
            echo '------------------ INICIANDO O SONAR'
            
            echo '------------------ FINALIZANDO O SONAR'
        }
        stage('MANUAL STEP'){
            echo '------------------ INICIANDO O MANUAL STEP'
            
            echo '------------------ FINALIZANDO O MANUAL STEP'
        }

        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    }
    /*
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {

            

        }

        stage('Push To Test Org') {
            
        }

        stage('Run Apex Test') {
            sh "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = sh returnStatus: true, script: "${toolbelt} force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') {
            //junit keepLongStdio: true, testResults: 'tests/* /*-junit.xml'
        }

        stage('Delete Test Org') {

        
    }
    }
    **/
}

def deployScratch(appKey, hubOrg, orgAlias, sfdcHost, jwtKeyFile){
    rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${appKey} --username ${hubOrg} --setalias ${orgAlias} --instanceurl ${sfdcHost} --setdefaultdevhubusername --jwtkeyfile ${jwtKeyFile} "
    if (rc != 0) { error 'HUB AUTH ERROR' }

    // need to pull out assigned username
    rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername --targetdevhubusername ${orgAlias}"
    printf rmsg
    def jsonSlurper = new JsonSlurperClassic()
    def robj = jsonSlurper.parseText(rmsg)
    if (robj.status != 0) { error 'org creation failed: ' + robj.message }
    SFDC_USERNAME=robj.result.username
    robj = null
}

def pushCodeScratchOrg(){
    rc = sh returnStatus: true, script: "${toolbelt} force:source:push --targetusername ${SFDC_USERNAME}"
    if (rc != 0) {
        error 'push failed'
    }
    // assign permset
    rc = sh returnStatus: true, script: "${toolbelt} force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
    if (rc != 0) {
        error 'permset:assign failed'
    }
}

def runApexTest(){
    sh "mkdir -p ${RUN_ARTIFACT_DIR}"
        timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'apex test run failed'
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
// executa os testes do LWC
def runLWCTests(){

}
// executa os testes do LC 
def runLCTests(){

}
// executa o logout antes de criar o pipeline 
def runLogoutSFDX(){

}
// deleta uma scratch org
def runDeleteScrathOrg(targetusername){
    timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:org:delete --targetusername ${targetusername} --noprompt"
            if (rc != 0) {
                error 'org deletion request failed'
            }
        }
}

