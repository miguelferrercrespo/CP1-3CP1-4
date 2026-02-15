pipeline {
	agent any
	
	options { skipDefaultCheckout() }
	
	stages {
		stage('Get Code') {
            steps {
				deleteDir()
				withCredentials([usernamePassword(credentialsId: 'github-credential', usernameVariable: 'GH_USER', passwordVariable: 'GH_PAT')]) {
                    sh '''
                        set -e
                        git clone --branch develop https://$GH_USER:$GH_PAT@github.com/miguelferrercrespo/CP1-3CP1-4 .
                    '''
                }
                stash name: 'code', includes: '**'
			}
		}
	}
}
