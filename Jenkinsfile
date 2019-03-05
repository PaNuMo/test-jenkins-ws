#!groovy​

import com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition

// This constant has to have as value the same value as the 
// first item of moduleOptions object in JenkinsfileOptions.json 
final ALL_MODULES = "All"

// The moduleOptions object from JenkinsfileOptions.json
def moduleOptions = {}

// Array of the module names in the order presented in
// moduleOptions object from JenkinsfileOptions.json
def moduleNames = []

// The environments object from JenkinsfileOptions.json
def serverOptions = {}

// Array of the server names in the order presented in
// environments object from JenkinsfileOptions.json
def serverNames = []

// Initialize global variables
node {
    // Checkout workspace here since we have to
    // get the init values from JenkinsfileOptions.json
    checkout([
        $class: 'GitSCM',
        branches: [[name: '*/master']],
        userRemoteConfigs: [[url: scm.getUserRemoteConfigs()[0].getUrl()]]
    ])

    def optionsJSON = readJSON file: 'JenkinsfileOptions.json'
    
    moduleOptions = optionsJSON.get("moduleOptions")
    moduleNames.addAll(moduleOptions.keySet())

    serverOptions = optionsJSON.get("environments")
    serverNames.addAll(serverOptions.keySet())
}

// Multiple select parameter definition using 'Extended Choice Parameter' plugin
def selectedModulesParam = new ExtendedChoiceParameterDefinition(
    "selectedModules", // Name
    "PT_CHECKBOX", // Choice type
    moduleNames.join(","), // Items
    "","","","","","","",
    "All", // Default Value
    "","","","","","","","","","","","","","","",false,false, 
    10, // Number of visible items
    "Which module(s) build/deploy?", // Description
    "," // Delimiter
);

// Set Pipeline parameters
properties([
    parameters([
        selectedModulesParam,
        choice(choices: serverNames, description: 'To which environment deploy?', name: 'environment'),  
        booleanParam(defaultValue: true, description: '', name: 'deployLatestTag'),
        string(defaultValue: '', description: 'Which version use?', name: 'artifactoryVersion')             
    ])
])

// Start Pipeline
pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    stages {
        stage('User Acceptance') {
            steps {
                script {  
                    def selectedModules = params.selectedModules.split(",")                    

                    // Build message to display in the summary for user acceptance
                    def userInputMessage = selectedModules[0] == ALL_MODULES ? "Deploying all modules" : "Deploying $selectedModules"
                    userInputMessage += " to ${params.environment}."
                    
                    def versionMessage = " From trunk."
                    if(params.deployLatestTag) {
                        versionMessage =  " Last version of each module."
                    }
                    else if (!isNullOrEmpty(params.artifactoryVersion)){
                        versionMessage =  " Version ${params.artifactoryVersion}."
                    }
                    userInputMessage += versionMessage

                    // Wait for 1 minute for user acceptance
                    timeout(time:1, unit:'MINUTES') {
                        userInput = input(
                            id: 'proceedInput', 
                            message: userInputMessage
                        )
                    }
                }
            }
        }

        stage('Checkout Module(s)') {
            steps {
                script {                   
                    def selectedModules = params.selectedModules.split(",")
                    
                    // If "All" checkbox was selected
                    if (selectedModules[0] == ALL_MODULES) {
                        // Checkout all
                        for (int i = 1; i < moduleNames.size(); i++) {
                            checkoutModule(moduleNames[i], moduleOptions, params.deployLatestTag, params.artifactoryVersion)
                        }
                    }
                    else {
                        // Checkout selected modules
                        for (int i = 0; i < selectedModules.size(); i++) {
                            def moduleName = selectedModules[i]
                            def moduleGitUrl = moduleOptions.get(moduleName)
                            checkoutModule(moduleName, moduleOptions, params.deployLatestTag, params.artifactoryVersion)
                        }
                    }                    
                }
	        }
	    }

        stage('Build') {
            steps {
                sh './gradlew clean deploy'
            }
        }

        stage('Deploy') {           
            steps {               
                withCredentials([usernamePassword(credentialsId: 'd2401c82-1cfc-4dc8-ae36-db88555ad209',
                    usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    script{
                        def serverNodes = serverOptions.get(params.environment)
                        for(int i = 0; i < serverNodes.size(); i++){
                            def node = serverNodes[i]
                            def nodePath = node.get("deployPath")
                            def nodeServer = node.get("server")
                            echo "Deploying to: $nodeServer"

                            //sh "scp -r bundles/osgi/modules $USERNAME@$nodeServer:$nodePath"
                        }
                    } 
                }  
            }
        }

        stage('Workspace Cleanup') {
            steps {
                deleteDir()
            }
        }
    }
}

/*
 * This method checkouts an specific module from VCS
 */
def checkoutModule(moduleName, moduleOptions, deployLatestTag, specificTag) {
    def checkoutUrl = moduleOptions.get(moduleName)

    if(checkoutUrl != null){
        def moduleTag = getModuleTag(moduleName, checkoutUrl, deployLatestTag, specificTag)
        def modulePath = "modules/${moduleName}"

        // If moduleTag is empty then checkout from trunk
        def moduleUrl = isNullOrEmpty(moduleTag) ? "${checkoutUrl}/trunk" : "${checkoutUrl}/${moduleTag}"
        
        checkout([
            $class: 'SubversionSCM', 
            locations: [[
                credentialsId: 'svn-server', 
                local: modulePath, 
                remote: moduleUrl
            ]]
        ])       
    }
    else {
        echo "ERROR Couldn't find a Git URL for $moduleName"
    }
}

/*
 * This method is used to get the module tag based on user input.
 */
def geModuleTag(moduleName, checkoutUrl, deployLatestTag, specificTag){
    
    def moduleTag = ""

    if (deployLatestTag) {
        withCredentials([usernamePassword(credentialsId: 'svn-server',
            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
            script{
                def tagTemp = sh(returnStdout: true, script: "svn log $checkoutUrl --limit 1 --non-interactive --no-auth-cache --username $USERNAME --password $PASSWORD | tail -n 2")
                moduleTag = tagTemp.substring(0, tagTemp.indexOf("\n"))     
            } 
        } 

        echo "The latest tag found for $moduleName is $moduleTag"
    }
    else if (!isNullOrEmpty(specificTag)){
        moduleTag = "tags/$specificTag"
    }

    return moduleTag
}

def isNullOrEmpty(someString){
    return !someString?.trim()
}