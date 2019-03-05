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

// The latest tag goten from one of the repositories
def tagVersion = ''

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
        stage('Checkout Module(s)') {
            steps {
                script {                   
                    // Checkout code from VCS
                    def selectedModules = params.selectedModules.split(",")
                    def allModulesSelected = selectedModules[0] == ALL_MODULES
                    
                    // If "All" checkbox was selected
                    if (allModulesSelected) {
                        // Checkout all
                        for (int i = 1; i < moduleNames.size(); i++) {
                            tagVersion = checkoutModule(moduleNames[i], moduleOptions, tagVersion)
                        }
                    }
                    else {
                        // Checkout selected modules
                        for (int i = 0; i < selectedModules.size(); i++) {
                            def moduleName = selectedModules[i]
                            def moduleGitUrl = moduleOptions.get(moduleName)
                            tagVersion = checkoutModule(moduleName, moduleOptions, tagVersion)
                        }
                    }

                    // Define message to display in the summary for user acceptance
                    def userInputMessage = allModulesSelected ? "Deploying all modules" : "Deploying $selectedModules"
                    userInputMessage += " to ${params.environment}. Version ${tagVersion}."

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
 * Checkout module
 * @param moduleName
 * @param moduleOptions
 * @param tagVersion
 * @return tagVersion
 */
def checkoutModule(moduleName, moduleOptions, tagVersion) {
    def checkoutUrl = moduleOptions.get(moduleName)
    def currentTag = ''
    def isTagVersionEmpty = isNullOrEmpty(tagVersion)

    if(checkoutUrl != null){                   
        def modulePath = "modules/${moduleName}"
        def moduleUrl = isNullOrEmpty(params.artifactoryVersion) ? "${checkoutUrl}/trunk" : "${checkoutUrl}/tags/${artifactoryVersion}"
        
        checkout([
            $class: 'SubversionSCM', 
            locations: [[
                credentialsId: 'svn-server', 
                local: modulePath, 
                remote: moduleUrl
            ]]
        ])

        if(isTagVersionEmpty){
            currentTag = sh(returnStdout: true, script: "cd $modulePath; git tag --sort version:refname | tail -1").trim()

            echo "Tag found: $currentTag"
        }
        else {
            echo "Tag version already exists: $tagVersion"
        }
    }
    else {
        echo "ERROR Couldn't find a Git URL for $moduleName"
    }

    return isTagVersionEmpty ? currentTag : tagVersion
}

def isNullOrEmpty(someString){
    return !someString?.trim()
}