def modulePath;

pipeline {
    agent any

    tools {
        jdk 'Jenkins_Java'
    }

    parameters {
        string(defaultValue: '/var/lib/jenkins/jenkins-ws', description: '', name: 'workspacePath')
        choice(
            choices: [
                'https://github.com/PaNuMo/test-module-one',
                'https://github.com/PaNuMo/test-module-two',
                'https://github.com/PaNuMo/test-module-three',
                'Build/Deploy All'
            ],
            description: 'Which module build/deploy?', name: 'moduleGitUrl')
        booleanParam(defaultValue: true, description: '', name: 'deployToServer')
    }

    stages {
        stage('Workspace Setup') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: env.GIT_URL]]
                ])
            }
        }

        stage('Checkout Module(s)') {
            steps {
                script {
                    def splittedUrl = params.moduleGitUrl.split('/')
                    modulePath = 'modules/' + splittedUrl[splittedUrl.length - 1]
                }

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: modulePath]],
                    userRemoteConfigs: [[url: params.moduleGitUrl]]
                ])
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
                sh 'cp -a bundles/osgi/modules/* /home/pnunez/Documents/Liferay/liferay-ce-portal-tomcat-7.1.2-ga3-20190107144105508/liferay-ce-portal-7.1.2-ga3/osgi/modules/'
                sh 'pwd'
            }
        }

        stage('Workspace Cleanup') {
            deleteDir()
        }
    }
}
