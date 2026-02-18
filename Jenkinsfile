pipeline {
	agent any

	triggers { githubPush() }

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
                        python3 -m flake8 --exit-zero --format=pylint src > flake8.out
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
                    sam deploy --config-env staging --resolve-s3 --no-fail-on-empty-changeset
                '''
            }
        }
		stage ('Rest Test') {
			steps {
				catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                    deleteDir()
                    unstash 'code'
                    sh '''
                        set -e
                        export BASE_URL="$(aws cloudformation describe-stacks \
                          --stack-name todo-list-aws-staging \
                          --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                          --output text)"
                        echo "BASE_URL=$BASE_URL"
                        python3 -m pytest test/integration/todoApiTest.py --junitxml=result-rest.xml
                    '''
					junit testResults: 'result-rest.xml', allowEmptyResults: true
                }
			}
		}
		stage('Promote') {
			when {
				expression { currentBuild.currentResult == 'SUCCESS' }
				}
			steps {
				deleteDir()
				withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GH_USER', passwordVariable: 'GH_PAT')]) {
					sh '''
						set -e
						git clone --branch master https://$GH_USER:$GH_PAT@github.com/miguelferrercrespo/CP1-3CP1-4.git .
						git config user.email "jenkins@local"
						git config user.name  "jenkins"
						git fetch origin develop
						git merge --no-ff origin/develop -m "Release"
						git push origin master
					'''
				}
			}
		}
	}
	 post {
        always {
            cleanWs()
        }
    }
}
