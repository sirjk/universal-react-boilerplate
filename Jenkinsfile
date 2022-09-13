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
                sh 'docker build -t builder:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-build'
                sh 'docker volume create vol-out'
                sh 'docker run --mount source=vol-out,destination=/outputVol builder:latest bash -c "cd .. && cp -r /universal-react-boilerplate /outputVol"'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
		            sh 'docker build -t tester:latest . -f docker-test'
            }
        }
        stage('Deploy') {
            steps {
		            echo 'Deploying..'
                sh 'docker build -t deploy:latest . -f /var/jenkins_home/workspace/DevOpsPipeline2/backend/docker-deploy'
                sh 'docker run --name deploy-container --mount source=vol-out,destination=/outputVol deploy:latest'
                sh 'rm -rf artifact'
		            sh 'mkdir artifact'
		            sh 'docker cp deploy-container:outputVol/shopBastim.jar ./artifact'
            }
        }
        stage('Publish') {
	          steps {
	            	echo 'Publishing..'
                script{
                    if(params.promote){
                        sh "mv ./artifact/shopBastim.jar ./artifact/shopBastim-${params.version}.jar"
                        archiveArtifacts artifacts: "artifact/shopBastim-${params.version}.jar"
                    }
                    else{
                        echo 'Pipeline finished work sucessfully but new version wasn\'t published.'
                    }
                }
            }
        }
    }
}
