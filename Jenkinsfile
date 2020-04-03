#!groovy
import groovy.json.JsonSlurperClassic
node {
    //default variables 
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
     def SFDC_DEVELOPER_MODE = "ORG"// ORG or PACKAGE
    //sfdx executable 
    def toolbelt = tool 'toolbelt'
    def SFDC_USERNAME
   

    //production variables 
    def HUB_ORG_USERNAME=env.HUB_ORG_DEVOPS_PRD
    def SFDC_HOST = env.SFDC_HOST_DEVOPS_PRD
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_PRD
    def ORG_ALIAS_DEVOPS_PRD = env.ORG_ALIAS_DEVOPS_PRD


    //sandbox variables or other environments (QA)
    def HUB_ORG_USERNAME_UAT =env.HUB_ORG_USERNAME_UAT
    //def SFDC_HOST_UAT = env.SFDC_HOST_DEVOPS_PRD
    //def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD // utilizado para todos os ambientes
    def CONNECTED_APP_CONSUMER_KEY_UAT =env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_UAT
    def ORG_ALIAS_DEVOPS_UAT = env.ORG_ALIAS_DEVOPS_UAT

    //sanbox variables (QA)
    def HUB_ORG_USERNAME_QA =env.HUB_ORG_USERNAME_QA
    //def SFDC_HOST_QA = env.SFDC_HOST_DEVOPS_QA
    //def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD // utilizado para todos os ambientes
    def CONNECTED_APP_CONSUMER_KEY_QA =env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_QA
    def ORG_ALIAS_DEVOPS_QA = env.ORG_ALIAS_DEVOPS_QA

    //dev variables or other environments (somente para desenvolvimento deste jenkinsfile)
    def HUB_ORG_USERNAME_DEV =env.HUB_ORG_USERNAME_DEV
    //def SFDC_HOST_UAT = env.SFDC_HOST_DEVOPS_PRD
    //def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD // utilizado para todos os ambientes
    def CONNECTED_APP_CONSUMER_KEY_DEV =env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_DEV
    def ORG_ALIAS_DEVOPS_DEV = env.ORG_ALIAS_DEVOPS_DEV


    stage('CLEAN WORKSPACE'){
        cleanWs()
        dir("${env.WORKSPACE}@tmp") {
            deleteDir()
        }
        dir("${env.WORKSPACE}@script") {
            deleteDir()
        }
        dir("${env.WORKSPACE}@script@tmp") {
            deleteDir()
        }
    }

    stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
    }

    if(env.BRANCH_NAME.startsWith('developer_')){
        stage('LOGOUT SFDX'){
            echo '------------------ INICIANDO O LOGOUT '
                try{
                    rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_DEV} -p"
                    if (rc != 0) { echo ' ERROR IN LOGOUT ' }
                }catch (all) {
                    echo all
                }
                
            echo '------------------ FINALIZANDO O LOGOUT '
        }

        stage ('ANALYSIS') {
            def mvnHome = tool 'mvn-default'

            sh "${mvnHome}/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs"

            def checkstyle = scanForIssues tool: checkStyle(pattern: '**/target/checkstyle-result.xml')
            publishIssues issues: [checkstyle]
   
            def pmd = scanForIssues tool: pmdParser(pattern: '**/target/pmd.xml')
            publishIssues issues: [pmd]
        
            def cpd = scanForIssues tool: cpd(pattern: '**/target/cpd.xml')
            publishIssues issues: [cpd]
        
            def spotbugs = scanForIssues tool: spotBugs(pattern: '**/target/findbugsXml.xml')
            publishIssues issues: [spotbugs]

            def maven = scanForIssues tool: mavenConsole()
            publishIssues issues: [maven]
        
            publishIssues id: 'analysis', name: 'All Issues', 
                issues: [checkstyle, pmd, spotbugs], 
                filters: [includePackage('io.jenkins.plugins.analysis.*')]
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY_DEV, HUB_ORG_USERNAME_DEV, ORG_ALIAS_DEVOPS_DEV, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('LIGHTNING COMPONENTS TESTS'){

                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests")
                }
                stage('GET RESULTS'){
                    junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
                }
                stage('DELETE SCRATCH ORG'){
                    echo '------------------ INICIANDO O SCRATCH ORG'
                    runDeleteScrathOrg(toolbelt,SFDC_USERNAME)
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
                try{
                    rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${ORG_ALIAS_DEVOPS_DEV} -p"
                    if (rc != 0) { echo ' ERROR IN LOGOUT ' }
                }catch (all) {
                    echo all
                }
                
            echo '------------------ FINALIZANDO O LOGOUT '
        }

        stage ('ANALYSIS') {
            def mvnHome = tool 'mvn-default'

            sh "${mvnHome}/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs"

            def checkstyle = scanForIssues tool: checkStyle(pattern: '**/target/checkstyle-result.xml')
            publishIssues issues: [checkstyle]
   
            def pmd = scanForIssues tool: pmdParser(pattern: '**/target/pmd.xml')
            publishIssues issues: [pmd]
        
            def cpd = scanForIssues tool: cpd(pattern: '**/target/cpd.xml')
            publishIssues issues: [cpd]
        
            def spotbugs = scanForIssues tool: spotBugs(pattern: '**/target/findbugsXml.xml')
            publishIssues issues: [spotbugs]

            def maven = scanForIssues tool: mavenConsole()
            publishIssues issues: [maven]
        
            publishIssues id: 'analysis', name: 'All Issues', 
                issues: [checkstyle, pmd, spotbugs], 
                filters: [includePackage('io.jenkins.plugins.analysis.*')]
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('DEPLOY IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY_DEV, HUB_ORG_USERNAME_DEV, ORG_ALIAS_DEVOPS_DEV, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests")
                }
                stage('GET RESULTS'){
                    junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
                }
                stage('DELETE SCRATCH ORG'){
                    echo '------------------ INICIANDO O SCRATCH ORG'
                    runDeleteScrathOrg(toolbelt,SFDC_USERNAME)
                    echo '------------------ FINALIZANDO O SCRATCH ORG'
                }
            }
            echo '------------------ FINALIZANDO O SCRATCH ORG'
        }

        stage('DEPLOY IN MERGE'){
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY_DEV, HUB_ORG_USERNAME_DEV, ORG_ALIAS_DEVOPS_DEV, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('LIGHTNING COMPONENTS TESTS'){

                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunAllTestsInOrg")
                }
                stage('GET RESULTS'){
                    junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
                }
                stage('DELETE SCRATCH ORG'){
                    echo '------------------ INICIANDO O SCRATCH ORG'
                    runDeleteScrathOrg(toolbelt,SFDC_USERNAME)
                    echo '------------------ FINALIZANDO O SCRATCH ORG'
                }

            }
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
}

def deployScratch(toolbelt,appKey, hubOrgUsername, orgAlias, sfdcHost, jwtKeyFile){
    rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${appKey} --username ${hubOrgUsername} --setalias ${orgAlias} --instanceurl ${sfdcHost} --setdefaultdevhubusername --jwtkeyfile ${jwtKeyFile} "
    if (rc != 0) { error 'HUB AUTH ERROR' }

    // need to pull out assigned username
    rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername --targetdevhubusername ${orgAlias}"
    echo rmsg
    def jsonSlurper = new JsonSlurperClassic()
    def robj = jsonSlurper.parseText(rmsg)
    if (robj.status != 0) { error 'org creation failed: ' + robj.message }
    return robj.result.username
}

def pushCodeScratchOrg(toolbelt,sfdcUsername){
    rc = sh returnStatus: true, script: "${toolbelt} force:source:push --targetusername ${sfdcUsername}"
    if (rc != 0) {
        error 'push failed'
    }
    // assign permset
    rc = sh returnStatus: true, script: "${toolbelt} force:user:permset:assign --targetusername ${sfdcUsername} --permsetname DreamHouse"
    if (rc != 0) {
        error 'permset:assign failed'
    }
}

//Values: 
//RunSpecifiedTests— > Only the tests that you specify are run.
//RunLocalTests—> All tests in your org are run, except the ones that originate from installed managed packages.
//RunAllTestsInOrg—> All tests are in your org and in installed managed packages are run.
def runApexTest(toolbelt, runArtifactDir, sfdcUsername, testLevel){
    sh "mkdir -p ${runArtifactDir}"
        timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:apex:test:run --testlevel  --outputdir ${runArtifactDir} --resultformat tap --targetusername ${sfdcUsername}"
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

// deleta uma scratch org
def runDeleteScrathOrg(toolbelt,targetusername){
    timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:org:delete --targetusername ${targetusername} --noprompt"
            if (rc != 0) {
                error 'org deletion request failed'
            }
        }
}

