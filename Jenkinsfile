pipeline {
    agent any
    
    parameters{
	  string(name:'version',defaultValue:'1.0.0', description:'The version of artifact')
	  booleanParam(name:'promote',defaultValue: false, description:'Publish new version')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker system prune --all --volumes -f'
                sh 'docker volume create --name vol-out'
                sh 'docker build -t builder:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-build'
                sh 'docker run --name build-container -v vol-out:/outputVol builder:latest'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
		            sh 'docker build -t tester:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-test'
                //sh 'docker run --mount source=vol-out,destination=/outputVol/ tester:latest'
            }
        }
        stage('Deploy') {
            steps {
		            echo 'Deploying..'
                sh 'docker build -t deploy:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-deploy'
                sh 'docker run --name deploy-container -v vol-out:/outputVol deploy:latest'
                sh 'rm -rf artifact'
		            sh 'mkdir artifact'
		            sh 'docker cp deploy-container:/outputVol/universal-react-boilerplate.tar.gz ./artifact'
            }
        }
        stage('Publish') {
	          steps {
	            	echo 'Publishing..'
                script{
                    if(params.promote){
                        sh "mv ./artifact/universal-react-boilerplate.tar.gz ./artifact/universal-react-boilerplate-${params.version}.tar.gz"
                        archiveArtifacts artifacts: "artifact/universal-react-boilerplate-${params.version}.tar.gz"
                    }
                    else{
                        echo 'Pipeline finished work sucessfully but new version wasn\'t published.'
                    }
                }
            }
        }
    }
}
