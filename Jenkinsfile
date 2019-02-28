#!groovy​

def modulesArray = [
    'https://github.com/PaNuMo/test-module-one',
    'https://github.com/PaNuMo/test-module-two',
    'https://github.com/PaNuMo/test-module-three',
    'Build/Deploy All'
]

node {
    checkout([
        $class: 'GitSCM',
        branches: [[name: '*/master']],
        userRemoteConfigs: [[url: env.GIT_URL]]
    ])

    def optionsJSON = readJSON file: 'JenkinsfileOptions.json'
    def moduleOptions = optionsJSON.get("moduleOptions")
    def moduleNames = moduleOptions.keySet()
    println(moduleOptions.keySet())
}

pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    parameters {
        choice(choices: moduleNames, description: 'Which module build/deploy?', name: 'moduleGitUrl')
        booleanParam(defaultValue: true, description: '', name: 'deployToServer')
    }

    stages {

        stage('Checkout Module(s)') {
            steps {
                script {
                    if (params.moduleGitUrl == modulesArray[modulesArray.size()-1]) {
                        for (int i = 0; i < modulesArray.size()-1; i++) {
                            def gitUrl = modulesArray[i];
                            def splittedUrl = gitUrl.split('/')
                            def modulePath = 'modules/' + splittedUrl[splittedUrl.length - 1]

                            println('Downloading from ' + gitUrl)
                            checkout([
                                $class: 'GitSCM',
                                branches: [[name: '*/master']],
                                extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
                                userRemoteConfigs: [[url: gitUrl]]
                            ])
                        }
                    }
                    else {
                        def splittedUrl = params.moduleGitUrl.split('/')
                        def modulePath = 'modules/' + splittedUrl[splittedUrl.length - 1]

                        println('Downloading from ' + params.moduleGitUrl)
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/master']],
                            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
                            userRemoteConfigs: [[url: params.moduleGitUrl]]
                        ])
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
