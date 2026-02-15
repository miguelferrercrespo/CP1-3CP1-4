pipeline {
	agent any
	
	options { skipDefaultCheckout() }
	
	stages {
		stage('Get Code') {
            steps {
				deleteDir()
				withCredentials([usernamePassword(credentialsId: 'github-pat',Variable: 'GH_PAT')]) {
                    sh '''
                        set -e
                        git clone --branch develop https://x-access-token:$GH_PAT@github.com/miguelferrercrespo/CP1-3CP1-4 .
                    '''
                }
                stash name: 'code', includes: '**'
			}
		}
	}
}
