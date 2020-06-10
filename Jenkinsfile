#!groovy

node {
    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def DEPLOYDIR='toDeploy'
    def TEST_LEVEL='RunLocalTests'

    def toolbelt = tool 'toolbelt'

    // -------------------------------------------------------------------------
    // Stage 1: Check out code from source control.
    // -------------------------------------------------------------------------
    stage('checkout source') {
        checkout scm
    }

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
     withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {

        // -------------------------------------------------------------------------
		// Stage 2: Authenticate to Salesforce using the jwt key.
		// -------------------------------------------------------------------------
		stage('Authorize to Salesforce') {
			rc = command "${toolbelt} sfdx force:auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY} --jwtkeyfile ${jwt_key_file} --username ${HUB_ORG}"
		    if (rc != 0) {
			    error 'Salesforce org authorization failed.'
		    }
		}

        // -------------------------------------------------------------------------
		// Stage 3: Prepare Deployment Files
		// -------------------------------------------------------------------------
		stage('Prepare Deployment Files') {
			rc = command "${toolbelt} sfdx force:source:convert -r force-app -d ${DEPLOYDIR}"
		    if (rc != 0) {
			    error 'Salesforce source convert failed.'
		    }
		}

        // -------------------------------------------------------------------------
		// Stage 4: Validate the Deployment
		// -------------------------------------------------------------------------
		stage('Validate Deployment') {
		    rc = command "${toolbelt} sfdx force:mdapi:deploy --checkonly -w 10 -d ${DEPLOYDIR} -l RunLocalTests -u ${HUB_ORG}"
		    if (rc != 0) {
		        error 'Salesforce deploy validation failed.'
		    }
		}

        // -------------------------------------------------------------------------
		// Stage 5: Perform the Deployment
		// -------------------------------------------------------------------------
		stage('Perform Deployment') {
		    rc = command "${toolbelt} sfdx force:mdapi:deploy -w 10 -d ${DEPLOYDIR} -lRunLocalTests -u ${HUB_ORG}"
		    if (rc != 0) {
		        error 'Salesforce deploy validation failed.'
		    }
		}
     }
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}