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

// Have selected modules in array instead of String 
// returned by ExtendedChoiceParameterDefinition
def selectedModules = []

// Check if 'All' modules option has been selected
def allModulesSelected = false

// Initialize global variables
node {
    // Checkout workspace here to get access to the options json.
    checkout scm

    def optionsJSON = readJSON file: 'JenkinsfileOptions.json'

    moduleOptions = optionsJSON.get("moduleOptions")
    moduleNames.addAll(moduleOptions.keySet())

    serverOptions = optionsJSON.get("environments")
    serverNames.addAll(serverOptions.keySet())

    selectedModules = params.selectedModules?.split(",")

    if (selectedModules) {
        allModulesSelected = selectedModules[0]  == ALL_MODULES
    }

    // Multiple select parameter definition using 'Extended Choice Parameter' plugin
    def selectedModulesParam = new ExtendedChoiceParameterDefinition(
            "selectedModules", // Name
            "PT_CHECKBOX", // Choice type
            moduleNames.join(","), // Items
            "","","","","","","",
            "", // Default Value
            "","","","","","","","","","","","","","","",false,false,
            10, // Number of visible items
            "Select modules to build/deploy.", // Description
            "," // Delimiter
    )

    // Set Pipeline parameters
    properties([
        parameters([
                selectedModulesParam,
                choice(choices: serverNames, description: 'Specify the target environment', name: 'environment'),
                booleanParam(defaultValue: false, description: '', name: 'deployLatestTag'),
                string(defaultValue: '', description: 'Specify a version to use', name: 'artifactoryVersion')
        ])
    ])
}


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
                    // Build message to display in the summary for user acceptance
                    def userInputMessage = allModulesSelected ? "Deploying all modules" : "Deploying $selectedModules"
                    userInputMessage += " to ${params.environment} "
                    
                    def versionMessage = "(from trunk)"
                    if(params.deployLatestTag) {
                        versionMessage =  "(last version of each module)"
                    }
                    else if (!isNullOrEmpty(params.artifactoryVersion)){
                        versionMessage =  "(version ${params.artifactoryVersion})"
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
                    // If "All" checkbox was selected
                    if (allModulesSelected) {
                        // Checkout all
                        for (int i = 1; i < moduleNames.size(); i++) {
                            checkoutModule(moduleNames[i], moduleOptions, params.deployLatestTag, params.artifactoryVersion)
                        }
                    }
                    else {
                        // Checkout selected modules
                        for(moduleName in selectedModules) {
                            def moduleGitUrl = moduleOptions.get(moduleName)
                            checkoutModule(moduleName, moduleOptions, params.deployLatestTag, params.artifactoryVersion)
                        }
                    }                    
                }
	        }
	    }

        stage('Build') {
            steps {
                sh './gradlew clean assemble'
            }
        }

        stage('Deploy') {           
            steps {     
                script {  
                    def serverNodes = serverOptions.get(params.environment)
                    for(int i = 0; i < serverNodes.size(); i++){
                        def node = serverNodes[i]
                        def nodePath = node.get("deployPath")
                        def nodeServer = node.get("server")
                        echo "Deploying to: $nodeServer"

                        // If "All" checkbox was selected
                        if (allModulesSelected) {
                            // Loop through all the modules, skip the first one since that's the 'All' option
                            for (int x = 1; x < moduleNames.size(); x++) {                               
                                sh "cp -a modules/${moduleNames[x]}/build/libs/* $nodePath"
                            }
                        }
                        else {
                            // Loop through selected modules
                            for(moduleName in selectedModules){
                                sh "cp -a modules/${moduleName}/build/libs/* $nodePath"
                            }
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
        def modulePath = "modules/${moduleName}"

        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/master']],
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
            userRemoteConfigs: [[url: checkoutUrl]]
        ])       
    }
    else {
        echo "ERROR Couldn't find a Git URL for $moduleName"
    }
}

def isNullOrEmpty(someString){
    return !someString?.trim()
}