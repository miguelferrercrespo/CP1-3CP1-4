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
                        git clone --branch master https://$GH_USER:$GH_PAT@github.com/miguelferrercrespo/CP1-3CP1-4 .
                    '''
                }
                stash name: 'code', includes: '**'
			}
		}
		stage('Deploy') {
            steps {
                deleteDir()
                unstash 'code'
                sh '''
                    set -e
                    sam build
                    sam deploy --config-env production --resolve-s3 --no-fail-on-empty-changeset
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
                          --stack-name todo-list-aws-production \
                          --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                          --output text)"
                        echo "BASE_URL=$BASE_URL"
                        python3 -m pytest test/integration/todoApiTest.py -k "test_api_listtodos or test_api_gettodo" --junitxml=result-rest.xml
                    '''
					junit testResults: 'result-rest.xml', allowEmptyResults: true
                }
			}
		}
	}
	 post {
        always 
		{ cleanWs() }
	}
}
