pipeline {
	agent any
	
	options { skipDefaultCheckout() }
	
	stages {
		stage('Get Code') {
            steps {
				deleteDir()
				withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GH_USER', passwordVariable: 'GH_PAT')]) {
                    sh '''
                        set -e
                        git clone --branch develop https://$GH_USER:$GH_PAT@github.com/miguelferrercrespo/CP1-3CP1-4 .
                    '''
                }
                stash name: 'code', includes: '**'
			}
		}
		
		stage ('Static Test') {
			steps {
				catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
					deleteDir()
					unstash 'code'
					
					sh '''

                        flake8 --exit-zero --format=pylint src > flake8.out

                        export PYTHONPATH=$WORKSPACE
                        bandit -r src -f custom --msg-template "{abspath}:{line}: [{test_id}] {msg}" > bandit.out

                        exit 0
					'''
					recordIssues tools: [
                        flake8(name: 'Flake8', pattern: 'flake8.out'),
                        pyLint(name: 'Bandit', pattern: 'bandit.out')
                    ]
					
				}			 
			}		
		}
		
		stage ('Deploy') {
            steps {
                deleteDir()
                unstash 'code'
                sh '''
                    set -e
                    sam build
                    sam deploy --config-env staging
                '''
            }
        }
	}
}
