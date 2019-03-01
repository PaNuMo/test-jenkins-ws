#!groovy​





com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition extendedChoiceParameterDefinition = new com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition(
    "OPTION_NAME", // name
    "PT_CHECKBOX", // type
    "option1,option2,option3", // values
    null, // projectName
    null, // propertyFile
    null, // groovyScript
    null, // groovyScriptFile
    null, // bindings
    null, // groovyClasspath
    null, // propertyKey
    "option1,option2", // defaultValue
    null, // defaultPropertyFile
    null, // defaultGroovyScript
    null, // defaultGroovyScriptFile
    null, // defaultBindings
    null, // defaultGroovyClasspath
    null, // defaultPropertyKey
    null, // descriptionPropertyValue
    null, // descriptionPropertyFile
    null, // descriptionGroovyScript
    null, // descriptionGroovyScriptFile
    null, // descriptionBindings
    null, // descriptionGroovyClasspath
    null, // descriptionPropertyKey
    null, // javascriptFile
    null, // javascript
    false, // saveJSONParameterToFile
    false, // quoteValue
    10, // visible item count
    "Select your options", // description
    "," //multiSelectDelimiter
)

def moduleOptions = {}
def moduleNames = []

def serverOptions = {}
def serverNames = []
def serverDeployPath = ''

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

pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    parameters {        
        choice(choices: moduleNames, description: 'Which module build/deploy?', name: 'moduleName')
        booleanParam(defaultValue: true, description: '', name: 'deployToServer')
        choice(choices: serverNames, description: 'To which server deploy?', name: 'serverName')
        extendedChoiceParameterDefinition
    }

    stages {

        stage('Checkout Module(s)') {
            steps {
                script {
                    if (params.moduleName == moduleNames[moduleNames.size()-1]) {
                        for (int i = 0; i < moduleNames.size()-1; i++) {
                            checkoutModule(moduleNames[i], moduleOptions)
                        }
                    }
                    else {
                        def moduleGitUrl = moduleOptions.get(params.moduleName)
                        checkoutModule(params.moduleName, moduleOptions)
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
            when {
                expression {
                    return params.deployToServer;
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

void checkoutModule(moduleName, moduleOptions) {
    def moduleGitUrl = moduleOptions.get(moduleName)
    def splittedUrl = moduleGitUrl.split('/')                    
    def modulePath = 'modules/' + splittedUrl[splittedUrl.length - 1]

    println('Start downloading module from ' + moduleGitUrl)
    checkout([
        $class: 'GitSCM',
        branches: [[name: '*/master']],
        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
        userRemoteConfigs: [[url: moduleGitUrl]]
    ])
}