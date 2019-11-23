pipeline {
	agent { label 'ansible' }

	options {
		buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '7'))
		ansiColor('xterm')
		timestamps()
		timeout(time: 10, unit: 'MINUTES')
	}

	triggers {
		pollSCM 'H/60 * * * *'
	}

	stages {
		stage('deploy:loadbalancer') {
			steps {

				// Encrypt remote password
				// ansibleVault action: 'encrypt_string', content: 'password', input: '', installation: 'ansible-bundled', output: '', vaultCredentialsId: 'ansible-vault-pass'

				// Deploy
				ansiblePlaybook installation: 'ansible-bundled', credentialsId: 'hetzner-ajourney-dfi-lb-staging', become:
				true , becomeUser: 'dev' ,
				vaultCredentialsId: 'ansible-vault-pass',
				disableHostKeyChecking: true ,
				colorized: true ,
				extras: "-vvvv" ,
				inventory:
				'ansible/hosts' ,
				playbook: 'ansible/ajourney-staging.yml'
			}
		}
	}
	post {
		success {
			slackSend(color: '#00FF00', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
		}

		failure {
			slackSend(color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

			emailext([subject           : '$DEFAULT_SUBJECT',
					  body              : '$DEFAULT_CONTENT',
					  recipientProviders: [
						  [$class: 'CulpritsRecipientProvider'],
						  [$class: 'DevelopersRecipientProvider'],
						  [$class: 'RequesterRecipientProvider']
					  ],
					  replyTo           : '$DEFAULT_REPLYTO',
					  to                : '$DEFAULT_RECIPIENTS'])
		}

		always {
			junit allowEmptyResults: true, testResults: '**/reports/junit/*.xml'

			// Clean workspace
			cleanWs()
		}

		changed {
			emailext([subject           : '$DEFAULT_SUBJECT',
					  body              : '$DEFAULT_CONTENT',
					  recipientProviders: [
						  [$class: 'CulpritsRecipientProvider'],
						  [$class: 'DevelopersRecipientProvider'],
						  [$class: 'RequesterRecipientProvider']
					  ],
					  replyTo           : '$DEFAULT_REPLYTO',
					  to                : '$DEFAULT_RECIPIENTS'])
		}
	}
}
