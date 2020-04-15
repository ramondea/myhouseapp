#!groovy
import groovy.json.JsonSlurperClassic
node {
    //default variables 
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_DEVELOPER_MODE = "ORG"// ORG or PACKAGE
    def PACKAGE_ALIAS = "myHouseApp"
    //sfdx executable 
    def toolbelt = tool 'toolbelt'
    def pmd = tool 'pmd'
    //variables for common use
    def SFDC_USERNAME
    def PACKAGE_VERSION
   
    //production variables (DEV HUB)
    def HUB_ORG_USERNAME=env.HUB_ORG_DEVOPS_PRD
    def SFDC_HOST = env.SFDC_HOST_DEVOPS_PRD
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DEVOPS_PRD
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DEVOPS_PRD
    def ORG_ALIAS_DEVOPS_PRD = env.ORG_ALIAS_DEVOPS_PRD


    //sandbox variables or other environments (UAT)
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

    //Limpando o workspace, sempre fazer isso pois o sfdx deixa alguns arquivos que dao erro em criar scratch orgs 
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
    //sempre utilizar esse comando em multibranch pipeline
    stage('CHECKOUT SOURCE') {
            echo '------------------ INICIANDO O CHECKOUT SOURCE '
            checkout scm
            echo '------------------ FINALIZANDO O CHECKOUT SOURCE '
    }
    //------------------------------
    //para cada padrao de branch é executado um pipeline diferente.
    //------------------------------
    
    if(env.BRANCH_NAME.startsWith('developer_')){
        
        // Analisa etapa de codigo estatica. 
        stage ('ANALYSIS') {
            
            sh "mkdir -p target"
            rc = sh returnStatus: true, script: "${pmd} pmd -d ./force-app/main/default/classes/ -f xml -language apex -R rulesets/apex/quickstart.xml -cache pmdcache -failOnViolation false -r ./target/pmd.xml"

            rpc = sh returnStatus: true, script: "${pmd} cpd --files ./force-app/main/default/classes/ --format xml --language apex --minimum-tokens 100 > ./target/cpd.xml"
   
            def pmdResult = scanForIssues tool: pmdParser(pattern: '**/target/pmd.xml')
            publishIssues issues: [pmdResult]
        
            def cpd = scanForIssues tool: cpd(pattern: '**/target/cpd.xml')
            publishIssues issues: [cpd]
        
            //def spotbugs = scanForIssues tool: spotBugs(pattern: '**/target/findbugsXml.xml')
            //publishIssues issues: [spotbugs]

            //def maven = scanForIssues tool: mavenConsole()
            //publishIssues issues: [maven]
        
            publishIssues id: 'analysis', name: 'All Issues', 
                issues: [pmdResult, cpd], 
                filters: [includePackage('io.jenkins.plugins.analysis.*')]
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '--------------------------------------// INICIANDO O LWC TESTS'
            
            echo '--------------------------------------//FINALIZANDO O LWC TESTS'
        }

        stage ('LIGHTNING LINT'){
            rmsg = sh returnStdout: true, script: "${toolbelt} force:lightning:lint force-app/main/default/aura --verbose --exit"
            echo rmsg
        }

        stage('VALIDATE IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY, HUB_ORG_USERNAME, ORG_ALIAS_DEVOPS_PRD, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('LIGHTNING COMPONENTS TESTS'){
                    
                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests", true)
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
            echo $BUILD_URL
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }

    }

    //na branch de release, serao executados os mesmos testes de DEV. 
    //etapa manual para instalacao do pacote no ambiente de merge. Nao gera versao ou promocao do pacote. 
    if(env.BRANCH_NAME.startsWith('release_')){
        stage ('ANALYSIS') {
            sh "mkdir -p target"
            rc = sh returnStatus: true, script: "${pmd} pmd -d ./force-app/main/default/classes/ -f xml -language apex -R rulesets/apex/quickstart.xml -cache pmdcache -failOnViolation false -r ./target/pmd.xml"

            rpc = sh returnStatus: true, script: "${pmd} cpd --files ./force-app/main/default/classes/ --format xml --language apex --minimum-tokens 100 > ./target/cpd.xml"
   
            def pmdResult = scanForIssues tool: pmdParser(pattern: '**/target/pmd.xml')
            publishIssues issues: [pmdResult]
        
            def cpd = scanForIssues tool: cpd(pattern: '**/target/cpd.xml')
            publishIssues issues: [cpd]
        
            //def spotbugs = scanForIssues tool: spotBugs(pattern: '**/target/findbugsXml.xml')
            //publishIssues issues: [spotbugs]

            //def maven = scanForIssues tool: mavenConsole()
            //publishIssues issues: [maven]
        
            publishIssues id: 'analysis', name: 'All Issues', 
                issues: [pmdResult, cpd], 
                filters: [includePackage('io.jenkins.plugins.analysis.*')]
        }

        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }

        stage('VALIDATE IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    // PRD SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY, HUB_ORG_USERNAME, ORG_ALIAS_DEVOPS_PRD, SFDC_HOST, jwt_key_file)
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY_DEV, HUB_ORG_USERNAME_DEV, ORG_ALIAS_DEVOPS_DEV, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests", false)
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
        // Etapa manual para que o aprovador possa publicar o pacote no ambiente de merge
        timeout(time: 10, unit: "HOURS") {
            input message: 'Approve Deploy?', ok: 'Yes'
            //SANDBOX DEVELOPER PRO
            stage('DEPLOY IN SANDBOX MERGE'){

                withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                    stage('DEPLOY SOURCE'){
                        deployMetadata(toolbelt,appKey, hubOrgUsername, orgAlias, sfdcHost, jwtKeyFile)
                    }
                    // uma boa pratica é rodar todos os testes para garantir que o novo codigo nao ira quebrar a aplicacao. 
                    // ** testes de regressao ** 
                    // devido a org esta quebrada, essa funcao sera desabilitada. 
                    stage('RUN ALL TESTS'){
                        //runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests",true)
                    }
                    stage('GET RESULTS'){
                        // junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
                    }  
               
                }
            }
        
        }
        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            echo $BUILD_URL
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    } 
    //na branch de uat, serao executados os mesmos testes de DEV.
    //etapa manual perguntando se o admin deseja gerar uma nova versao e instalar o pacote. 
    if(env.BRANCH_NAME.startsWith('uat')){
        stage ('ANALYSIS') {
            sh "mkdir -p target"
            rc = sh returnStatus: true, script: "${pmd} pmd -d ./force-app/main/default/classes/ -f xml -language apex -R rulesets/apex/quickstart.xml -cache pmdcache -failOnViolation false -r ./target/pmd.xml"

            rpc = sh returnStatus: true, script: "${pmd} cpd --files ./force-app/main/default/classes/ --format xml --language apex --minimum-tokens 100 > ./target/cpd.xml"
   
            def pmdResult = scanForIssues tool: pmdParser(pattern: '**/target/pmd.xml')
            publishIssues issues: [pmdResult]
        
            def cpd = scanForIssues tool: cpd(pattern: '**/target/cpd.xml')
            publishIssues issues: [cpd]
        
            //def spotbugs = scanForIssues tool: spotBugs(pattern: '**/target/findbugsXml.xml')
            //publishIssues issues: [spotbugs]

            //def maven = scanForIssues tool: mavenConsole()
            //publishIssues issues: [maven]
        
            publishIssues id: 'analysis', name: 'All Issues', 
                issues: [pmdResult, cpd], 
                filters: [includePackage('io.jenkins.plugins.analysis.*')]
        }
        stage ('LIGHTNING WEB COMPONENTS TESTS'){
            echo '------------------ INICIANDO O LWC TESTS'
            
            echo '------------------ FINALIZANDO O LWC TESTS'
        }
        stage('VALIDATE IN SCRATCH ORG'){
            echo '------------------ INICIANDO O SCRATCH ORG'
            withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                stage('CREATE SCRATCH ORG'){
                    // PRD SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY, HUB_ORG_USERNAME, ORG_ALIAS_DEVOPS_PRD, SFDC_HOST, jwt_key_file)
                    SFDC_USERNAME = deployScratch(toolbelt,CONNECTED_APP_CONSUMER_KEY_DEV, HUB_ORG_USERNAME_DEV, ORG_ALIAS_DEVOPS_DEV, SFDC_HOST, jwt_key_file)
                    echo SFDC_USERNAME
                }
                stage('PUSH CODE'){
                    pushCodeScratchOrg(toolbelt,SFDC_USERNAME)
                }
                stage('APEX TEST'){
                    runApexTest(toolbelt, RUN_ARTIFACT_DIR, SFDC_USERNAME, "RunLocalTests", false)
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
        // Etapa manual para que o aprovador possa publicar o pacote no ambiente de merge
        timeout(time: 10, unit: "HOURS") {
            input message: 'Generate Package Version?', ok: 'Yes'
                withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                    stage('GENERATE PACKAGE VERSION'){
                        rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG_USERNAME} --setalias ${ORG_ALIAS_DEVOPS_PRD} --instanceurl ${SFDC_HOST} --setdefaultdevhubusername --jwtkeyfile ${jwt_key_file} "
                        if (rc != 0) { 
                            error 'ORG AUTH ERROR'
                        }
                        //gerando uma nova versao do pacote: 
                        //verificar o funcionamento da versao do pacote, este comando altera o package.json
                        output = sh returnStdout: true, script: "${toolbelt}/sfdx force:package:version:create --package ${PACKAGE_ALIAS} --installationkeybypass --wait 10 --json --targetdevhubusername ${HUB_ORG_USERNAME}"
                        // aguarda 5 minutos 
                        sleep 300

                        def jsonSlurper = new JsonSlurperClassic()
                        def response = jsonSlurper.parseText(output)
                        //vou precisar deste cara mais tarde
                        PACKAGE_VERSION = response.result.SubscriberPackageVersionId

                        response = null

                        echo ${PACKAGE_VERSION}
                    }
                    stage('INSTALL PACKAGE VERSION'){
                        //conectar na org de destino para instalar o pacote 
                        withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                            rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY_DEVOPS_UAT} --username ${HUB_ORG_USERNAME_UAT} --setalias ${ORG_ALIAS_DEVOPS_UAT} --instanceurl ${SFDC_HOST_UAT} --jwtkeyfile ${jwt_key_file} "
                            if (rc != 0) { 
                                error 'ORG AUTH ERROR'
                            }
                        }
                        rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:package:install --package ${PACKAGE_VERSION} --targetusername ${HUB_ORG_USERNAME_UAT}  --wait 10"
                    }

                }
        }
        stage('NOTIFICATION'){
            echo '------------------ INICIANDO O NOTIFICATION'
            echo $BUILD_URL
            echo '------------------ FINALIZANDO O NOTIFICATION'
        }
    }
    
    if(env.BRANCH_NAME.startsWith('master')){
        
    }
}

/** 
* 
*/
def deployMetadata(toolbelt,appKey, hubOrgUsername, orgAlias, sfdcHost, jwtKeyFile){
    //force:auth:sfdxurl:store mais simples a utilizacao
    rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${appKey} --username ${hubOrgUsername} --setalias ${orgAlias} --instanceurl ${sfdcHost} --setdefaultdevhubusername --jwtkeyfile ${jwtKeyFile} "
    if (rc != 0) { 
        error 'ORG AUTH ERROR'
    }
    // deploy source 
    rDeploy = sh returnStdout: true, script: "${toolbelt} force:source:deploy --targetusername ${hubOrgUsername} --sourcepath /force-app --testlevel RunLocalTests --wait 10 --json"
    echo rDeploy
    def jsonSlurper = new JsonSlurperClassic()
    def robj = jsonSlurper.parseText(rmsg)
    if (robj.status != 0) { 
        error 'org creation failed: ' + robj.message 
    }
           
}

def deployPackage(toolbelt,appKey, hubOrgUsername, orgAlias, sfdcHost, jwtKeyFile){

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
def runApexTest(toolbelt, runArtifactDir, sfdcUsername, testLevel, saveData){
    if(saveData){
        sh "mkdir -p ${runArtifactDir}"
        timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:apex:test:run --testlevel  --outputdir ${runArtifactDir} --resultformat tap --targetusername ${sfdcUsername}"
            if (rc != 0) {
                error 'apex test run failed'
            }
        }
    }else{
        sh "mkdir -p ${runArtifactDir}"
        timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:apex:test:run --testlevel  --resultformat tap --targetusername ${sfdcUsername}"
            if (rc != 0) {
                error 'apex test run failed'
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

// deleta uma scratch org
def runDeleteScrathOrg(toolbelt,targetusername){
    timeout(time: 120, unit: 'SECONDS') {
            rc = sh returnStatus: true, script: "${toolbelt} force:org:delete --targetusername ${targetusername} --noprompt"
            if (rc != 0) {
                error 'org deletion request failed'
            }
        }
}
// efetua logout de uma sessao sfdx 
def logout(toolbelt, orgAliasDevHub){
    try{
        rc = sh returnStatus: true, script: "${toolbelt} force:auth:logout --targetusername ${orgAliasDevHub} -p"
        if (rc != 0) { 
            echo ' ERROR IN LOGOUT ' 
        }
    }catch (all) {
        echo all
    }
}

