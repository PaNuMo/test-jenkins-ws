#!groovy​

import com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition

def moduleOptions = {}
def moduleNames = []

def serverOptions = {}
def serverNames = []
def serverDeployPath = ''

def tagVersion = ''

node {
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
    "","","","","","","","","","","","","", 
    "","","","","","","","","","",false,false, 
    10, // Number of visible items
    "Which module(s) build/deploy?", // Description
    "," // Delimiter
);

def selectCheckoutParam = new ExtendedChoiceParameterDefinition(
    "selectedCheckout", // Name
    "PT_RADIO", // Choice type
    "Git,Artifactory", // Items
    "","","","","","","",
    "Git", // Default Value
    "","","","","","","","","","","","","","","",false,false, 
    2, // Number of visible items
    "From where checkout the modules?", // Description
    "," // Delimiter
);




properties([
    parameters([
        selectCheckoutParam,
        selectedModulesParam,
        booleanParam(defaultValue: false, description: '', name: 'uploadToArtifactory'),
        booleanParam(defaultValue: true, description: '', name: 'deployToServer'),
        choice(choices: serverNames, description: 'To which server deploy?', name: 'serverName'),        
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
                    
                    if(params.selectedCheckout == "Git"){
                        // Checkout code from Git
                        println("Checkout source code from Git")
                        def selectedModules = params.selectedModules.split(",")
                        if (selectedModules[0] == moduleNames[0]) {
                            // Checkout all
                            for (int i = 1; i < moduleNames.size(); i++) {
                                checkoutModule(moduleNames[i], moduleOptions, tagVersion)
                            }
                        }
                        else {
                            // Checkout selected modules
                            for (int i = 0; i < selectedModules.size(); i++) {
                                def moduleName = selectedModules[i]
                                def moduleGitUrl = moduleOptions.get(moduleName)
                                checkoutModule(moduleName, moduleOptions, tagVersion)
                            }
                        }
                    }
                    else{
                        // Get jars from Artifactory
                        println("Download jars from Artifactory")
                        println("which version???")

                    }
                }
	        }
	    }

        stage('Build') {
            when {
                expression {
                    return params.selectedCheckout == "Git"
                }
            }
            steps {
                sh './gradlew clean deploy'
            }
        }

        stage('Upload to Artifactory') {
            when {
                expression {
                    return params.uploadToArtifactory
                }
            }
            steps {
                sh './gradlew clean deploy'
            }
        }

        stage('Deploy') {
            when {
                expression {
                    return params.deployToServer
                }
            }

            steps {
                script{
                    serverDeployPath = serverOptions.get(params.serverName)
                }
                sh "cp -a bundles/osgi/modules/* $serverDeployPath"
            }
        }

        stage('Workspace Cleanup') {
            steps {
                deleteDir()
            }
        }
    }
}

void checkoutModule(moduleName, moduleOptions, tagVersion) {
    def moduleGitUrl = moduleOptions.get(moduleName)
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
            println('*** Getting TAG version')
            dir(modulePath)(
                def currentTag = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
                println("TAG: " + currentTag)
            )
        }
    }
    else {
        println("ERROR: Couldn't find a Git URL for " + moduleName)
    }
}