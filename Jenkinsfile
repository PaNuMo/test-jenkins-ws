
pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    environment {
        MODULE_PATH = "modules/three"
    }

    parameters {
        string(defaultValue: '/var/lib/jenkins/jenkins-ws', description: '', name: 'workspacePath')
        choice(choices: [
                'https://github.com/PaNuMo/test-module-one',
                'https://github.com/PaNuMo/test-module-two',
                'https://github.com/PaNuMo/test-module-three'],
            description: 'Which module build/deploy?', name: 'moduleGitUrl')
        booleanParam(defaultValue: true, description: '', name: 'deployToServer')
    }

    stages {
        stage('Workspace Setup') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: params.workspacePath]],
                    userRemoteConfigs: [[url: env.GIT_URL]]
                ])
            }
        }

        stage('Checkout Module') {
            steps {
                script {
                    println('************************')

                    println(MODULE_PATH)

                    def urlArray = params.moduleGitUrl.split('/')
                    println(urlArray.length)
                }

                dir(params.workspacePath) {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/master']],
                        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: env.MODULE_PATH]],
                        userRemoteConfigs: [[url: params.moduleGitUrl]]
                    ])
                }
	          }
	      }

        stage('Build') {
            steps {
                dir(params.workspacePath) {
                    sh './gradlew clean deploy'
                }
            }
        }

        stage('Deploy') {
            when {
                expression {
                    return params.deployToServer;
                }
            }

            steps {
                sh 'cp -a /var/lib/jenkins/jenkins-ws/bundles/osgi/modules/* /home/pnunez/Documents/Liferay/liferay-ce-portal-tomcat-7.1.2-ga3-20190107144105508/liferay-ce-portal-7.1.2-ga3/osgi/modules/'
            }
        }
    }
}
