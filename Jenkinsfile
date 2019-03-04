#!groovy​

import com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition

def moduleOptions = {}
def moduleNames = []

def serverOptions = {}
def serverNames = []
def serverDeployPath = ''

def tagVersion = ''

node {
    //def scmUrl = scm.getUserRemoteConfigs()[0].getUrl()
    echo GIT_URL
    echo '$GIT_URL'

    checkout([
        $class: 'GitSCM',
        branches: [[name: '*/master']],
        userRemoteConfigs: [[url: 'https://github.com/PaNuMo/test-jenkins-ws']]
    ])

    def optionsJSON = readJSON file: 'JenkinsfileOptions.json'
    
    moduleOptions = optionsJSON.get("moduleOptions")
    moduleNames.addAll(moduleOptions.keySet())

    serverOptions = optionsJSON.get("environments")
    serverNames.addAll(serverOptions.keySet())
}

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

properties([
    parameters([
        selectedModulesParam,
        choice(choices: serverNames, description: 'To which environment deploy?', name: 'environment'),  
        booleanParam(defaultValue: true, description: '', name: 'deployLatestTag'),
        string(defaultValue: '', description: 'Which version use?', name: 'artifactoryVersion')             
    ])
])


pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    stages {

        stage('Checkout Module(s)') {
            steps {
                script {
                    
                    // Checkout code from Git
                    def selectedModules = params.selectedModules.split(",")
                    if (selectedModules[0] == moduleNames[0]) {
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

                    def userInputMessage = (selectedModules[0] == moduleNames[0]) ? "Deploying all modules" : "Deploying " + selectedModules
                    userInputMessage += " to " + params.environment + ". Version " + tagVersion + "."

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
                        sh 'echo $USERNAME'
                        echo "password is $PASSWORD"

                        def serverNodes = serverOptions.get(params.environment)
                        for(int i = 0; i < serverNodes.size(); i++){
                            def node = serverNodes.get(i)
                            def nodePath = node.get("path")
                            def nodeServer = node.get("server")
                            println("Deploying to: " + nodeServer)
                            
                            //sh "cp -a bundles/osgi/modules/* $nodePath"

                            sh "scp -r bundles/osgi/modules $USERNAME@$nodeServer:$nodePath"
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

String checkoutModule(moduleName, moduleOptions, tagVersion) {
    def moduleGitUrl = moduleOptions.get(moduleName)
    def currentTag = ''
    if(moduleGitUrl != null){
        def splittedUrl = moduleGitUrl.split('/')                    
        def modulePath = 'modules/' + splittedUrl[splittedUrl.length - 1]

        println('Start downloading module from ' + moduleGitUrl)
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/master']],
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
            userRemoteConfigs: [[url: moduleGitUrl]]
        ])

        if(tagVersion == ''){
            currentTag = sh(returnStdout: true, script: "cd $modulePath; git tag --sort version:refname | tail -1").trim()
            println("Tag found: " + currentTag)
        }
        else {
            println("Tag version already exists: " + tagVersion)
        }
    }
    else {
        println("ERROR: Couldn't find a Git URL for " + moduleName)
    }

    return tagVersion != '' ? tagVersion : currentTag
}