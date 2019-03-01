#!groovy​



import com.cwctravel.hudson.plugins.extended_choice_parameter.ExtendedChoiceParameterDefinition

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

def extendedChoiceParam = new ExtendedChoiceParameterDefinition("name", 
            "PT_CHECKBOX", 
            "blue,green,yellow,blue,black", 
            "PipelineProject",
            "", 
            "",
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "blue,green,yellow,blue,black", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            "", 
            false,
            false, 
            3, 
            "select something", 
            ",");


properties(
    [parameters([
        choice(choices: moduleNames, description: 'Which module build/deploy?', name: 'moduleName'),
        booleanParam(defaultValue: true, description: '', name: 'deployToServer'),
        choice(choices: serverNames, description: 'To which server deploy?', name: 'serverName'),
        extendedChoiceParam
    ]
        )]
)


pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    // parameters {        
    //     choice(choices: moduleNames, description: 'Which module build/deploy?', name: 'moduleName')
    //     booleanParam(defaultValue: true, description: '', name: 'deployToServer')
    //     choice(choices: serverNames, description: 'To which server deploy?', name: 'serverName')

    // } 

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
    }
    else {
        println("ERROR: Couldn't find a Git URL for " + moduleName)
    }
}