
pipeline {
    agent any

    environment {
        IMAGE_NAME = '441870321480.dkr.ecr.us-east-1.amazonaws.com/test-image-vuln-scan-base'
        IMAGE_TAG = 'latest'
	GIT_COMMIT_HASH = sh (script: "git rev-parse --short `git log -n 1 --pretty=format:'%H'`", returnStdout: true)
	GIT_COMMITER = sh (script: "git show -s --pretty=%an", returnStdout: true)
    }

    stages {
	stage ('Start') {
	      steps {
		// send build started notifications
		      slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}] by ${env.GIT_COMMITER} #${env.GIT_COMMIT_HASH}'  (${env.BUILD_URL})")
	      }
	}
        stage('Prepare') {
            steps {
                echo 'Initializing submodules..'
                sh 'ln -s local.env .env'
                sh 'git submodule update --init --recursive'
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker build . -t ${IMAGE_NAME}:ci'
		sh 'docker push ${IMAGE_NAME}:ci'
            }
        }
        stage('Scan') {
            steps {
		echo 'Scanning..'
//                sh 'curl -s https://ci-tools.anchore.io/inline_scan-v0.3.3 | bash -s -- ${IMAGE_NAME}:ci'
		writeFile file: "anchore_images", text: "${IMAGE_NAME}:ci"
	        anchore name: "anchore_images"
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
//                junit '**/target/*.xml'
            }
        }
        stage('QA') {
            steps {
                echo 'Checking code..'
            }
        }

        stage('Push Image') {
            steps {
            	sh '$(aws ecr get-login --no-include-email)'
		sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${IMAGE_TAG}'
                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
		sh 'docker tag ${IMAGE_NAME}:ci ${IMAGE_NAME}:${GIT_COMMIT_HASH}'
                sh 'docker push ${IMAGE_NAME}:${GIT_COMMIT_HASH}'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
    post {
	    success {
	      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

	      emailext (
		  subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
		  body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
		    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
		  recipientProviders: [[$class: 'DevelopersRecipientProvider']]
		)
	    }
	    failure {
	      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
	    }
    }
}
