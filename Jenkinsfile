#!groovy​

def modulesArray = [
    'https://github.com/PaNuMo/test-module-one',
    'https://github.com/PaNuMo/test-module-two',
    'https://github.com/PaNuMo/test-module-three',
    'Build/Deploy All'
]

def moduleOptions = null
def moduleNames = null

node {
    checkout([
        $class: 'GitSCM',
        branches: [[name: '*/master']],
        userRemoteConfigs: [[url: env.GIT_URL]]
    ])

    def optionsJSON = readJSON file: 'JenkinsfileOptions.json'
    moduleOptions = optionsJSON.get("moduleOptions")
    moduleNames = moduleOptions.keySet().join('\n')
    println(moduleNames.getClass())
}

pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    parameters {
        choice(choices: moduleNames, description: 'Which module build/deploy?', name: 'moduleName')
        booleanParam(defaultValue: true, description: '', name: 'deployToServer')
    }

    stages {

        stage('Checkout Module(s)') {
            steps {
                script {
                    if (params.moduleGitUrl == moduleNames[moduleNames.size()-1]) {
                        for (int i = 0; i < moduleNames.size()-1; i++) {
                            checkoutModule(moduleNames[i])
                        }
                    }
                    else {
                        def moduleGitUrl = moduleOptions.get(moduleName)
                        checkoutModule(moduleGitUrl)
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
                sh 'cp -a bundles/osgi/modules/* /home/pnunez/Documents/Liferay/DEV-liferay-ce-portal-7.1.2-ga3/osgi/modules/'
            }
        }

        stage('Workspace Cleanup') {
            steps {
                deleteDir()
            }
        }
    }
}

void checkoutModule(moduleGitUrl) {
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